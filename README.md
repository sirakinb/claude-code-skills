# Claude Code Skills

A collection of custom skills for Claude Code.

## Skills

| Skill | Description |
|-------|-------------|
| **build-feature** | Autonomous task loop that picks ready tasks, implements them, updates progress.txt, commits, and repeats |
| **compound-engineering** | Compound Engineering workflow following the Plan → Work → Review → Compound loop |
| **dev-browser** | Browser automation with persistent page state for testing and web interactions |
| **pdf** | Comprehensive PDF manipulation toolkit for extracting, creating, and processing PDFs |
| **prd** | Generate Product Requirements Documents (PRDs) for new features |
| **ralph** | Set up Ralph for autonomous feature development with task dependencies |

## Installation

To use these skills with Claude Code, copy the skill folders to your Claude Code skills directory:

```bash
cp -r <skill-name> ~/.claude/skills/
```

## Usage

Each skill can be invoked by its trigger phrases. For example:
- `/pdf` - Work with PDF files
- `/prd` - Create a product requirements document
- `/ralph` - Set up autonomous feature development
- "build feature" or "run the loop" - Start the build feature loop
- "plan this feature" or "compound learnings" - Use compound engineering workflow
- "go to [url]" or "test the website" - Use browser automation
