---
name: remove-ai-code-slop
description: Scan the current branch's diff and remove AI-generated code slop — unnecessary comments, gratuitous defensive checks, `any` casts, and style inconsistencies with the surrounding code. Use this skill whenever the user wants to clean up AI-generated code, mentions "slop", asks to make AI code look more human-written, or wants to polish a branch before review. Also trigger when the user says things like "clean up this branch", "remove the AI smell", or "make this code less obviously AI-written".
license: MIT
metadata:
  author: Diego Petrucci
  version: "1.0"
---

# Remove AI Code Slop

Clean up a branch by removing the telltale signs of AI-generated code. The goal is to make the diff look like a human wrote it — matching the style, density, and conventions already present in the codebase.

## On trigger

1. Determine the base branch. Use `git merge-base` with the default branch (check `main`, then `master`, then fall back to `origin/HEAD`) to find the common ancestor.
2. Get the diff: `git diff <base>...HEAD --name-only` to list changed files, then read each one.
3. Ask the user: **"Should I go ahead and make edits directly, or would you prefer I show you what I'd change first?"**
4. Proceed accordingly.

## What to look for

Work file by file through the diff. For each file, read the full file (not just the diff) so you understand the existing style and conventions. Then look for these patterns **only in the lines introduced by this branch**:

### Unnecessary comments
Comments that state the obvious, restate the code in English, or explain standard library usage. The key test: look at the rest of the file — if existing code at similar complexity doesn't have comments, the new code shouldn't either. Remove the comment, not the code.

**Example:**
```
// Check if user is authenticated before proceeding
if (!user.isAuthenticated) {
```
If the surrounding codebase doesn't comment trivial conditionals like this, remove the comment.

### Gratuitous defensive checks
Extra try/catch blocks, null guards, or validation that the surrounding code doesn't use — especially when the caller already guarantees the precondition. AI models tend to add safety nets "just in case" even when the architecture makes them redundant.

**Example:**
```
function processOrder(order: Order) {
  if (!order) throw new Error('Order is required');  // caller already validates
  try {
    // ...entire function body wrapped in try/catch for no reason
  } catch (e) {
    throw e;  // catch-and-rethrow adds nothing
  }
}
```
If existing functions in the file trust their callers, the new code should too.

### `any` casts to sidestep types
Casts to `any` (or `as any`) that exist only to make the compiler stop complaining rather than fixing the actual type issue. If the rest of the codebase properly types things, fix the type rather than removing the cast — but if fixing the type is non-trivial, flag it for the user rather than guessing.

### Style inconsistencies
Anything that sticks out compared to the rest of the file: different naming conventions, different error handling patterns, different import styles, different levels of abstraction. The branch's code should blend in seamlessly with what was already there.

## How to make changes

- If the user chose **silent editing**: make all edits, then report a 1-3 sentence summary of what you changed.
- If the user chose **review first**: for each file, briefly list what you'd change and why (grouped, not line-by-line). Wait for the user's go-ahead before editing.

In both cases, end with a short summary (1-3 sentences) of the overall changes.

## Important constraints

- Only touch lines introduced in this branch. Never "improve" pre-existing code that wasn't part of the diff.
- When in doubt, leave it. The goal is to remove things that are clearly AI slop, not to do a full code review.
- Don't introduce new changes, refactors, or "improvements" — only remove or simplify.
