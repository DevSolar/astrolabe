---
name: code-skeleton
description: Use this skill to "provide a code skeleton" for a class, group of functions, or other small-to-mid-sized collection of source code when the purpose of that code has been sufficiently determined.
---

# Code Skeleton Skill

## Preconditions

- The current working directory should be the root directory of a tool's repository.

- You should know the version control system in use. (Check for Git, Subversion, Mercurial, RCS. If you can identify neither, ask the user which VCS is being used.)

- There should be no uncommitted modifications (`git status -s` or equivalent). If there are, ask the user to clean or stash the repository first.

- You need to know the global use case for the tool in question. If you haven't already read them, use an existing `GEMINI.md` and/or `README.md` to get a first idea, then check the tool's interactive help (`<tool> --help`) for orientation on the expected functionality.

- Inspect existing project files to mirror the codebase's naming conventions, prefixes, header guard style, and code formatting.

- If not already clear from the query, ask the user to set the scope of the skeleton. This should usually be a single function, class, or implementation file. Warn the user if their request is not specific enough to give a good result.

## Skeleton and Comment-Driven Development

Based on the user's stated scope, and following the style already established in the project, create the code skeleton (if not already existing):

- **Header file**: Include guards, function declarations, and comprehensive documentation comments (detailing parameters, return values, and failure modes).

- **Implementation file**: Include the header, function definitions with matching signatures, valid dummy returns, and `(void)param;` statements for unused parameters to ensure the code compiles cleanly. Inside each function body, add structured comments outlining the core points of the implementation to allow for easy orientation when implementing the logic.

- **Test file skeleton**: Outline the expected test scenarios, register test stubs in the suite, and suggest any necessary mocking.
