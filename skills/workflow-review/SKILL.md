---
name: workflow-review
description: Use this skill to audit a tool with regard to its workflow, i.e. running use cases and identifying shortcomings in functionality, edge-case handling, and user experience.
---

# Workflow Review Skill

## Preconditions

- The current work directory should be the root directory of the tool's repository.

- You should know the version control system in use. (Check for Git, Subversion, Mercurial, RCS. If you can identify neither, ask the user which VCS is being used.)

- There should be no uncommited modifications. ('git status -s' or equivalent.) If there are, ask the user to clean the repository first.

- You need to know the global use case for the tool in question. If you haven't already read them, use an existing GEMINI.md and / or README.md to get a first idea, then try to get the tool's interactive help ('<tool> --help') for a first orientation of the tool's expected functionality.

- If not already clear from the query, ask the user to set the scope of the review, i.e. which workflows to review specifically. Warn the user if their request is not specific enough and would involve more than a dozen or so different use cases.

- Based on the user's stated scope, create a list of use cases (different ways the workflow might go). Start with the most common behavior ("sunny path"), then define the various failure modes (failures, aborts, etc.). Provide the list as artifact.

- Present that list to the user, and converse about whether it's what the user intended. Amend the list based on the user's feedback, until the user confirms the list.

## Reviewing a use case

- If necessary, set up test data for the planned use case review.

- Run the tool as outlined by the use case.

- Check the results for unexpected behavior.

- Mirror the tool's output to the user (or, if the output was significant, point the user to the name of the file where you stored the output). Report any unexpected behavior.

- Retrieve the user's feedback whether they are satisfied with the results. If they are not, come to an understanding what the desired results should be.

- Compile a TODO list detailing which changes need to be made for which use case.

## Finalizing the Review

- Complete the TODO list and provide it as an artifact.

- Only *after* the review list has been finished can you offer to fix the issues on the TODO list.
