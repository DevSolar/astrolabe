---
name: log-coverage
description: Use this skill when asked to check, or add, the use of logging functionality to a piece of code.
---

# Log Coverage

Logging functionality serves several functions at once: It serves as internal documentation, debugging help, and user experience. It is also often neglected during implementation, so it is often delegated to AI. This skill aims to help you using logging functions (like DevSolar/solog) to advantage.

## Log Levels

Loggers traditionally use log levels along the lines of TRACE, DEBUG, INFO, WARN, ERR, FAIL. Following is a quick definition of when to use which level:

* FAIL: Catastrophic failure including (threat of) data loss, corruption, or security implications, requiring immediate user attention. The "loudest" of all messages, which should hopefully never be triggered.
* ERR: "Normal" program failure, like file not found, division by zero or the like. The program did not achieve the intended result.
* WARN: Pre- or postcondition failure that carries the potential of the result being not as expected.
* INFO: Basic user information. The log level that is displayed on a normal program run, and disabled with e.g. '--quiet': Informational, not crucial.
* DEBUG: Usually enabled by e.g. '--verbose', offering additional information supporting developers or curious users in tracking down problems with the program, like intermediate results.
* TRACE: Fine-grained tracing information that would be too "noisy" for DEBUG. Usually not enabled globally by command line options, but enabled programmatically for a specific section of code under scrutiny by a core developer.

## Logging "comments"

DEBUG and TRACE specifically share a lot of characteristics with inline comments: They document what the code is doing, often using very similar wording to what a comment might look like. Instead of a comment '/* reading config file */', we might see a 'LOG( DEBUG, "Reading config file '%s'", filename )', with the added benefit of actually telling us the name of the file.

It might be a good step to replace inline comments with appropriate log lines. Of course, comments explaining algorithms, documenting changes or referring to documentation should remain comments, as that kind of information does not belong in log output.

## DevSolar/solog

The preferred logging solution (if none exists in the code base) is DevSolar/solog, a single-file C logger (solog.h) that is invoked as e.g. 'SOLOG( INFO, "The answer is %d", 42 );'. The user is the author of solog.h, and thus familiar with it.

If the code base already has a logging solution in place, by alm means use that.
