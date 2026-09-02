---
name: reworking-gim
description: >-
  Cheatsheet for future sessions working on the gim codebase. Covers
  architecture, established error-handling patterns, known gotchas (Unicode
  quotes, pull-context detection), and a backlog of remaining improvements
  discovered during the September 2026 error-handling pass.
---

# Reworking gim — Agent Cheatsheet

`gim` is a monolithic Perl porcelain for git.
Source: `/data/data/com.termux/files/home/repos/gim/gim` (~5700 lines).

---

## Architecture Quick Reference

| Concept | Details |
|---|---|
| Structure | Single Perl file; every subcommand is a `package` that extends `worker` |
| Entry point | Bottom of file: `util::subcommand` → `$obj->init()` → `$obj->run()` |
| Config | `Config::IniFiles` in `$cfg`; keys via `$cfg->val('section', 'key')` |
| Colour | `Term::ANSIColor` via `fmt::*` helpers (`fmt::hash`, `fmt::name`, `fmt::warning`, …) |
| i18n | `util::localize( 'Template [_1] text', $arg )` — Locale::Maketext style |
| Errors | `util::throw( util::localize( '...' ) )` dies; caught at top level and printed |
| Warnings | `warn( fmt::warning( ... ) )` — yellow, non-fatal |

### Key packages (top of file → subcommands)

```
util        — rc_check, throw, localize, filespec_check, subcommand, …
fmt         — coloured output helpers
dbg         — debug tracing (enter/leave/msg)
executor    — open3 wrapper; executor::run + executor->new (streaming)
logger      — extends executor; formats git log output
worker      — base class; init_git_dir, init_status, init_history, init_pager, …
<subcommand>— one package per gim subcommand, e.g. package clone { … }
```

---

## Error-Handling Pattern (established September 2026)

### Pattern A — simple executor::run call
```perl
my ( $out, $err, $rc ) = executor::run( \@command, throw_on_fail => 0 );

if ( $rc != 0 )
{
    if ( $err =~ /specific pattern/i )
    {
        util::throw( util::localize( 'Short crisp message.' ) );
    }
    else
    {
        util::throw( util::localize( 'Generic failure label.' ) . "\n" . $err . "\n" );
    }
}
```

### Pattern B — streaming executor->new + close_exe
```perl
my $exe = executor->new( \@command );

while ( $exe->readln() ) { … }

local $@ = '';
eval { $exe->close_exe(); };

if ( $@ )
{
    my $err = $exe->{ errors } // $@;    # $exe->{ errors } is private slot
    if ( $err =~ /pattern/i ) { util::throw( util::localize( '...' ) ); }
    else                      { die $@; }   # re-throw unknown errors
}
```

### Pattern C — cross-package blessed call
```perl
bless( $self, 'stash' );
local @ARGV = ( 'push', … );
local $@ = '';
eval { $self->run(); };
if ( $@ ) { util::throw( util::localize( 'Could not save changes …' ) . "\n" . $@ ); }
```

---

## Known Gotchas

### Unicode curly quotes in string literals
Many existing strings use fancy curly quotes (Unicode U+2018/U+2019/U+201C/U+201D).
These break `replace_file_content` because the search string must match byte-for-byte.
**Use Python patching for any block containing curly quotes:**
```bash
python3 - << 'PYEOF'
content = open('gim', 'r', encoding='utf-8').read()
lines = content.split('\n')
# Replace lines[N-1] through lines[M-1] (0-indexed)
new_block = "...replacement..."
lines[N-1:M] = new_block.split('\n')
open('gim', 'w', encoding='utf-8').write('\n'.join(lines))
PYEOF
```

### `rebase::start` pull-context detection (fragile)
`pull::run` injects `--quiet` into `@ARGV` when calling `rebase::start`.
`start` detects this to suppress its own diagnostics (pull handles them itself):
```perl
my $from_pull = grep { /^--quiet$/ } @ARGV;
if ( $rc != 0 && ! $from_pull ) { … }
```
This breaks if a user passes `--quiet` directly to `gim rebase`.
A cleaner fix would be a method argument or a blessed flag — deferred.

### `throw_on_fail => 0` without an rc check
Some pre-existing calls use `throw_on_fail => 0` but never inspect `$rc`.
After any refactor, grep for `throw_on_fail => 0` and confirm every occurrence
either checks `$rc` or has a comment explaining the silence is intentional.

### `$exe->{ errors }` is a private slot
`executor` stores stderr in `$self->{ errors }` inside `close_exe`. There is no
public accessor. Reading it directly is the established pattern for Pattern B,
but it could break if `executor` is ever refactored.

### `diff --prev` bug (FIXED September 2026)
The original code used a coderef callback that read the outer `$prev` (always
`undef`) instead of the option value. Fixed by switching to a scalar ref:
```perl
my $prev;
::GetOptions( 'prev|p:1' => \$prev );
if ( defined $prev ) { unshift( @ARGV, "HEAD~$prev" ); }
```
Documented here as a reminder of the coderef-vs-scalar-ref trap in GetOptions.

---

## Remaining Improvement Backlog

Ordered roughly by priority (highest first).

---

### 1. `stash::push` — no user-visible error handling

`stash::push` (around line 4350) calls `executor::run` with the default
`throw_on_fail => 1`, so a direct `gim stash push` failure falls through to the
generic `rc_check` message ("Failure while executing 'git stash'…").

**Fix:** Apply Pattern A; map these patterns:
- `No local changes to save` → "Nothing to stash."
- `locked` / `index.lock` → "Index is locked; another git process may be running."

---

### 2. `revert` — imprecise restore after reset

The comment "A better choice would be checking @ARGV for files that are still
tracked" remains valid. After `git reset --`, some files are no longer in the
index, so `git restore` fails for them — currently silently swallowed. The ideal
fix eliminates the silence by only calling restore on files still tracked:

```perl
# After git reset -- @ARGV:
my @tracked_cmd = ( 'git', 'ls-files', '--', @ARGV );
my ( $ls_out ) = executor::run( \@tracked_cmd );
my @tracked = split( "\n", $ls_out );
if ( @tracked )
{
    my @restore_cmd = ( 'git', 'restore', '--', @tracked );
    executor::run( \@restore_cmd );    # can now use throw_on_fail => 1
}
```

---

### 3. `rmbranch` — force-delete without merge warning

Remote branch deletes (`git push --delete`) and local force deletes
(`git branch -D`) run unconditionally when `--force` is passed, with no
warning that the branch may contain unmerged work.

**Fix:** For local deletes, attempt `git branch -d` (safe) first. If it exits
non-zero with "not fully merged", warn the user and require explicit `--force`
to proceed with `git branch -D`.

---

### 4. `commit` — editor-crash vs user-abort ambiguity

If the interactive editor itself crashes (e.g. SIGKILL), git exits 130 and
`$err` may be empty, falling through to the generic "Commit failed." message.

**Fix:** Add before the generic fallback:
```perl
elsif ( $rc == 130 )
{
    util::throw( util::localize( 'Commit aborted.' ) );
}
```

---

### 5. `pull` — stash pop conflict: stash entry not named in warning

When `stash::pop` during a pull encounters conflicts, the warning tells the user
to "resolve with 'gim resolve'" but does not name the stash entry. Since the
push during pull is always `stash@{0}`, a targeted message would be clearer.
Very low impact; the entry name is stable in this context.

---

## Style Notes

- Error messages: short imperative sentence + `'gim <subcommand>'` hint where
  applicable. No trailing period after a `'gim cmd'` example.
- Successful operations: intentionally silent — no "Done." confirmation,
  consistent with git's own behaviour.
- `fmt::warning()` is yellow. `util::throw` uses red output internally.
- All user-visible strings go through `util::localize()`, even when it is
  currently a no-op, to enable future translation.
