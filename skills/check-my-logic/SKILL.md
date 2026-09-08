---
name: check-my-logic
description: Use this skill to perform a thorough logic inspection and sanity check on source code to catch common oversights such as off-by-one errors, missing safety/null checks, resource leaks, edge-case failures, type overflow/underflow, and unhandled error returns.
---

# Check My Logic Skill

Perform a focused, fine-grained inspection of source code to catch base mistakes and common developer oversights before runtime execution or code review.

## Preconditions

- Identify the specific target code scope (a function, file, module, diff, or uncommitted change).
- Ensure complete context: inspect full implementations and schemas (avoid snippet tunnel vision; read surrounding context and header/type definitions if needed).
- Identify the programming language, memory management paradigm (manual, RAII, garbage-collected), and concurrency model.

## Inspection Checklist

Walk through the code logic line-by-line and branch-by-branch against the following categories:

### 1. Boundary & Off-By-One Errors
- **Loop Bounds**: Verify loop start/end conditions (`<` vs `<=`, `0` to `N-1` vs `1` to `N`).
- **Array / Buffer Indexing**: Check boundary indices, slicing bounds, and buffer sizes.
- **Empty / Edge Inputs**: Inspect behavior on zero-length arrays, empty strings, null pointers, single-element inputs, or boundary values (0, MAX_INT).

### 2. Resource Management & Lifetime
- **Allocation / Deallocation**: Confirm that resources (file descriptors, sockets, memory buffers, database handles, mutex locks) are released/closed on **all** exit paths (success, failure, early return, exceptions).
- **RAII / Cleanups**: Ensure proper use of language cleanup constructs (`defer`, `finally`, `using`, RAII guards, or explicit teardowns).
- **Double Release / Use-After-Free**: Check for multiple frees, dangling pointers, or operations on closed resources.

### 3. Safety, Nullability & Validation
- **Null / Nil / Undefined Checks**: Verify pointers, optionals, or references are checked before dereferencing.
- **Key / Lookup Safety**: Ensure map/dictionary lookups check for key existence before accessing values or handle missing keys gracefully.
- **Input Guarding**: Check that public methods and internal utilities validate caller assumptions or arguments.

### 4. Control Flow & Boolean Logic
- **Condition Inversions**: Check for boolean logic errors (e.g. `!A || !B` vs `!(A && B)`).
- **Switch / Match Fallthrough**: Verify all branches end with explicit return/break or deliberate fallthrough annotations.
- **State Mutation Consistency**: Ensure shared or persistent state is not mutated before validating preconditions that could abort execution.

### 5. Error & Exception Handling
- **Return Status Verification**: Confirm that all fallible system calls, file operations, network requests, or helper function return codes are checked.
- **Swallowed Errors**: Ensure catch/except/error blocks handle or log errors rather than silently ignoring failures.
- **Partial Failure Rollbacks**: Verify that partial operations roll back or clean up state if a mid-procedure step fails.

### 6. Numeric Precision & Types
- **Overflow / Underflow**: Check for numeric arithmetic that could exceed data type limits or wrap around unexpectedly.
- **Division by Zero**: Verify denominators are non-zero before division/modulo operations.
- **Signed vs Unsigned**: Check type conversions, cast safety, and comparison between signed and unsigned types.
- **Float Comparison**: Ensure floating-point equality isn't checked using direct `==`.

## Execution Workflow

1. **Scope Definition**: Locate and read the complete target code file or function.
2. **Path Analysis**: Identify all entry points, happy paths, early returns, error paths, and exit points.
3. **Checklist Walkthrough**: Systematically evaluate each path against the 6 checklist categories.
4. **Report Findings**: Present findings clearly to the user using the following format:
   - **File & Location**: `path/to/file:L<start>-L<end>`
   - **Category**: (e.g. *Off-By-One*, *Resource Leak*, *Missing Null Guard*)
   - **Description**: Concise explanation of the defect or oversight.
   - **Impact**: Potential runtime failure, crash, or unexpected behavior.
   - **Recommended Fix**: Code snippet showing the minimal correction needed.
