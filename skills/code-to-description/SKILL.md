---
name: code-to-description
description: Use this skill to deconstruct legacy, prototype, or dense code into a concise, high-level, declarative description. Relieves the developer of reading, abstracting, and documenting legacy mechanics so they can focus immediately on organic, maintainable reimplementation.
---

# Code-to-Description Skill

## Purpose & Philosophy

The goal of this skill is to bridge the gap between **existing code** and **clean, organic reimplementation**.

When refactoring or rewriting legacy, dense, or prototype code, reading low-level C boilerplate (memory management, string juggling, cleanup paths, repetitive error checks) creates cognitive friction.

This skill extracts the **essence and domain logic** of existing functions and captures it in a **concise, high-level, declarative description** placed directly inside function stubs. This allows the developer to immediately implement a clean, modern, and idiomatic solution without getting bogged down in legacy details.

---

## Target Abstraction Level

The description must be **declarative, concise, and focused on domain rules**, not a line-by-line procedural translation or pseudocode. Aim for brevity—typically 3 to 10 lines of comment per function.

### What to Capture:
- Basic functionality: How inputs map to outputs and side effects.
- Non-obvious invariants: Essential domain rules or format assumptions not obvious from the signature.

### What to Omit (Do NOT Include):
- **Defensive Boilerplate**: Standard `NULL` checks, empty string guards, or trivial parameter validation.
- **Memory & Resource Management**: Temporary buffers, allocation, copying, cleanup, and freeing logic.
- **Trivial Execution Mechanics**: Standard system return code checking, obvious error handling etc.
- **Verbose Logging Transcriptions**: Verbatim log strings.

---

## Delivery Workflow

1. **Locate Source & Target**: Identify the original functions to analyze and the target file/stub to update.
2. **Abstract Logic**: Strip out legacy boilerplate (memory handling, error bubbling, trivial guards) and extract the pure domain rules, fallback chains, and mappings.
3. **Embed in Compilable Stubs**:
   - Create function stubs matching the target codebase's naming conventions and signatures.
   - Include minimal scaffolding (`(void)param;`, dummy returns) so the build succeeds.
   - Embed the concise declarative comment block directly inside the function body.
