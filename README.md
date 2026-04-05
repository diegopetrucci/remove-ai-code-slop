# Remove AI Code Slop

An [agent skill](https://agentskills.io/home) that scans your branch's diff and removes telltale signs of AI-generated code.

## What it does

AI models tend to leave fingerprints in code: unnecessary comments, gratuitous defensive checks, `any` casts, and style that doesn't match the surrounding codebase. This skill finds and removes them so your code looks like a human wrote it. It:

- Checks the diff of your current branch against the base branch
- Reads each changed file in full to understand existing conventions
- Removes or simplifies AI slop patterns only in lines introduced by the branch
- Optionally shows you proposed changes before editing

## Installation

```bash
npx skills add https://github.com/diegopetrucci/remove-ai-code-slop --skill remove-ai-code-slop
```

## Usage

Trigger the skill while on a feature branch:

- **Claude Code:** `/remove-ai-code-slop` or say "clean up AI slop" / "remove AI code smell"

The skill will ask whether you want silent edits or a review-first workflow, then clean up the branch.

## License

MIT
