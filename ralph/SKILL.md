---
name: ralph
description: "Autonomous feature development - setup and execution. Triggers on: ralph, set up ralph, run ralph, run the loop, implement tasks. Three modes: (1) New Feature - plan and create tasks for a single feature (2) Existing Tasks - set up Ralph for existing prd.json (3) Full Product - plan and create tasks for entire product with multiple features."
---

# Ralph Feature Setup

Interactive feature planning that creates ralph-ready tasks in prd.json format.

---

## FIRST: Ensure Ralph Files Exist

Before doing anything else, check if the ralph execution files exist. If not, create them.

### Check for files:

```bash
ls scripts/ralph/ralph.sh scripts/ralph/prompt.md 2>/dev/null
```

### If files don't exist, create them:

```bash
mkdir -p scripts/ralph
```

**Create `scripts/ralph/ralph.sh`:**

```bash
cat > scripts/ralph/ralph.sh << 'RALPH_SCRIPT'
#!/bin/bash
# Ralph - Long-running AI agent loop
# Usage: ./ralph.sh [max_iterations]

set -e

MAX_ITERATIONS=${1:-10}
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PRD_FILE="$SCRIPT_DIR/prd.json"
PROGRESS_FILE="$SCRIPT_DIR/progress.txt"
ARCHIVE_DIR="$SCRIPT_DIR/archive"
LAST_BRANCH_FILE="$SCRIPT_DIR/.last-branch"

# Archive previous run if branch changed
if [ -f "$PRD_FILE" ] && [ -f "$LAST_BRANCH_FILE" ]; then
  CURRENT_BRANCH=$(jq -r '.branchName // empty' "$PRD_FILE" 2>/dev/null || echo "")
  LAST_BRANCH=$(cat "$LAST_BRANCH_FILE" 2>/dev/null || echo "")

  if [ -n "$CURRENT_BRANCH" ] && [ -n "$LAST_BRANCH" ] && [ "$CURRENT_BRANCH" != "$LAST_BRANCH" ]; then
    DATE=$(date +%Y-%m-%d)
    FOLDER_NAME=$(echo "$LAST_BRANCH" | sed 's|^ralph/||')
    ARCHIVE_FOLDER="$ARCHIVE_DIR/$DATE-$FOLDER_NAME"

    echo "Archiving previous run: $LAST_BRANCH"
    mkdir -p "$ARCHIVE_FOLDER"
    [ -f "$PRD_FILE" ] && cp "$PRD_FILE" "$ARCHIVE_FOLDER/"
    [ -f "$PROGRESS_FILE" ] && cp "$PROGRESS_FILE" "$ARCHIVE_FOLDER/"
    echo "   Archived to: $ARCHIVE_FOLDER"

    echo "# Ralph Progress Log" > "$PROGRESS_FILE"
    echo "Started: $(date)" >> "$PROGRESS_FILE"
    echo "---" >> "$PROGRESS_FILE"
  fi
fi

# Track current branch
if [ -f "$PRD_FILE" ]; then
  CURRENT_BRANCH=$(jq -r '.branchName // empty' "$PRD_FILE" 2>/dev/null || echo "")
  if [ -n "$CURRENT_BRANCH" ]; then
    echo "$CURRENT_BRANCH" > "$LAST_BRANCH_FILE"
  fi
fi

# Initialize progress file if it doesn't exist
if [ ! -f "$PROGRESS_FILE" ]; then
  echo "# Ralph Progress Log" > "$PROGRESS_FILE"
  echo "Started: $(date)" >> "$PROGRESS_FILE"
  echo "---" >> "$PROGRESS_FILE"
fi

echo "Starting Ralph - Max iterations: $MAX_ITERATIONS"

for i in $(seq 1 $MAX_ITERATIONS); do
  echo ""
  echo "═══════════════════════════════════════════════════════"
  echo "  Ralph Iteration $i of $MAX_ITERATIONS"
  echo "═══════════════════════════════════════════════════════"

  OUTPUT=$(cat "$SCRIPT_DIR/prompt.md" | claude --dangerously-skip-permissions 2>&1 | tee /dev/stderr) || true

  if echo "$OUTPUT" | grep -q "<promise>COMPLETE</promise>"; then
    echo ""
    echo "Ralph completed all tasks!"
    echo "Completed at iteration $i of $MAX_ITERATIONS"
    exit 0
  fi

  echo "Iteration $i complete. Continuing..."
  sleep 2
done

echo ""
echo "Ralph reached max iterations ($MAX_ITERATIONS) without completing all tasks."
echo "Check $PROGRESS_FILE for status."
exit 1
RALPH_SCRIPT

chmod +x scripts/ralph/ralph.sh
```

**Create `scripts/ralph/prompt.md`:**

```bash
cat > scripts/ralph/prompt.md << 'RALPH_PROMPT'
# Ralph Agent Instructions

You are an autonomous coding agent working on a software project.

## Your Task

1. Read the PRD at `prd.json` (in the same directory as this file)
2. Read the progress log at `progress.txt` (check Codebase Patterns section first)
3. Check you're on the correct branch from PRD `branchName`. If not, check it out or create from main.
4. Pick the **highest priority** user story where `passes: false`
5. Implement that single user story
6. Run quality checks (e.g., typecheck, lint, test - use whatever your project requires)
7. Update AGENTS.md files if you discover reusable patterns (see below)
8. If checks pass, commit ALL changes with message: `feat: [Story ID] - [Story Title]`
9. Update the PRD to set `passes: true` for the completed story
10. Append your progress to `progress.txt`

## Progress Report Format

APPEND to progress.txt (never replace, always append):
```
## [Date/Time] - [Story ID]
Session: [current session reference]
- What was implemented
- Files changed
- **Learnings for future iterations:**
  - Patterns discovered (e.g., "this codebase uses X for Y")
  - Gotchas encountered (e.g., "don't forget to update Z when changing W")
  - Useful context (e.g., "the evaluation panel is in component X")
---
```

The learnings section is critical - it helps future iterations avoid repeating mistakes.

## Consolidate Patterns

If you discover a **reusable pattern**, add it to the `## Codebase Patterns` section at the TOP of progress.txt:

```
## Codebase Patterns
- Example: Use `sql<number>` template for aggregations
- Example: Always use `IF NOT EXISTS` for migrations
```

Only add patterns that are **general and reusable**, not story-specific details.

## Update AGENTS.md Files

Before committing, check if edited files have learnings worth preserving in nearby AGENTS.md files:

1. Identify directories with edited files
2. Check for existing AGENTS.md in those directories
3. Add valuable learnings future developers/agents should know

**Good AGENTS.md additions:**
- "When modifying X, also update Y to keep them in sync"
- "This module uses pattern Z for all API calls"

**Do NOT add:** Story-specific details, temporary notes, info already in progress.txt

## Quality Requirements

- ALL commits must pass quality checks (typecheck, lint, test)
- Do NOT commit broken code
- Keep changes focused and minimal
- Follow existing code patterns

## Browser Testing (Required for Frontend Stories)

For any story that changes UI:
1. Load the `dev-browser` skill
2. Navigate to the relevant page
3. Verify the UI changes work as expected

A frontend story is NOT complete until browser verification passes.

## Stop Condition

After completing a user story, check if ALL stories have `passes: true`.

If ALL stories are complete, reply with:
<promise>COMPLETE</promise>

If there are still stories with `passes: false`, end normally (another iteration will continue).

## Important

- Work on ONE story per iteration
- Commit frequently
- Keep CI green
- Read Codebase Patterns in progress.txt before starting
RALPH_PROMPT
```

**Confirm files were created:**

```bash
ls -la scripts/ralph/
```

---

## The Job

**Three modes:**

### Mode 1: New Feature (Single Feature)
1. Chat through the feature - Ask clarifying questions
2. Break into small user stories - Each completable in one iteration
3. Create prd.json - Stories with `passes: false`
4. Initialize progress.txt

### Mode 2: Existing Tasks
1. Verify prd.json exists and is valid
2. Show current status - Which stories pass/fail
3. Set up progress.txt if needed

### Mode 3: Full Product Build (Multiple Features)
1. Chat through the product - Ask about features, scope, requirements
2. Break ALL features into small user stories - Same sizing rules as Mode 1
3. Create prd.json with all stories - Ordered by dependency (schema → backend → UI)
4. Initialize progress.txt
5. Ralph works through all stories until entire product is complete

**Ask the user which mode they need:**
```
Are you:
1. Starting a new feature (single feature - I'll help you plan and create tasks)
2. Using existing tasks (I'll set up Ralph to run your prd.json)
3. Building a full product (multiple features - I'll help you plan the entire product)
```

---

## Step 1: Understand What You're Building

### For Mode 1 (Single Feature):
```
What feature are you building?
```

Then ask clarifying questions:
- What's the user-facing goal?
- What parts of the codebase will this touch? (database, UI, API, etc.)
- Are there any existing patterns to follow?
- What should it look like when done?

### For Mode 3 (Full Product):
```
What product are you building? Tell me about:
- The core purpose / problem it solves
- The main features you want
- Any technical constraints (stack, existing code, etc.)
```

Then work through each feature:
- What are the key user stories?
- What's the MVP scope vs nice-to-have?
- How do features relate to each other?

**Keep asking until you have enough detail to break it into stories.**

---

## Step 2: Break Into User Stories

**Each story must be completable in ONE Ralph iteration (~one context window).**

Ralph spawns a fresh Claude instance per iteration with no memory of previous work. If a story is too big, the LLM runs out of context before finishing.

### Right-sized stories:
- Add a database column + migration
- Create a single UI component
- Implement one server action
- Add a filter to an existing list
- Write tests for one module

### Too big (split these):
- "Build the entire dashboard" → Split into: schema, queries, components, filters
- "Add authentication" → Split into: schema, middleware, login UI, session handling
- "Refactor the API" → Split into one story per endpoint

**Rule of thumb:** If you can't describe the change in 2-3 sentences, it's too big.

---

## Step 3: Order by Dependencies

Stories execute in priority order. Earlier stories must complete before later ones can use their output.

**Typical order:**
1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Integration / E2E tests

For Mode 3 (Full Product), chain features together:
```
Feature 1 stories (priority 1-4)
Feature 2 stories (priority 5-8) - can depend on Feature 1
Feature 3 stories (priority 9-12) - can depend on Features 1 & 2
```

---

## Step 4: Create prd.json

Save to `scripts/ralph/prd.json`:

```json
{
  "project": "[Project Name]",
  "branchName": "ralph/[feature-name-kebab-case]",
  "description": "[Feature/Product description]",
  "userStories": [
    {
      "id": "US-001",
      "title": "[Story title - action-oriented]",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": [
        "Criterion 1 - specific and verifiable",
        "Criterion 2",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

---

## Step 5: Initialize progress.txt

Create `scripts/ralph/progress.txt`:

```bash
cat > scripts/ralph/progress.txt << 'EOF'
# Ralph Progress Log
Started: [current date]
Feature: [feature/product name]

## Codebase Patterns
(Patterns discovered during this build)

---
EOF
```

---

## Step 6: Confirm Setup

```
✅ Ralph is ready!

**Feature:** [name]
**Branch:** ralph/[feature-name]

**User Stories:**
1. US-001: [title] - priority 1
2. US-002: [title] - priority 2
...

**Files created:**
- scripts/ralph/ralph.sh (execution loop)
- scripts/ralph/prompt.md (agent instructions)
- scripts/ralph/prd.json (your stories)
- scripts/ralph/progress.txt (iteration memory)

**To run Ralph:**
./scripts/ralph/ralph.sh [max_iterations]

Ralph will work through each story in priority order until all pass.
```

---

## Mode 2: Setting Up Existing Tasks

If the user already has prd.json:

### 1. Verify prd.json exists:

```bash
cat scripts/ralph/prd.json | jq '.userStories | length'
```

### 2. Show current status:

```
**Current Status:**
- ✅ Passing: [N] stories
- ❌ Not passing: [M] stories

**Next story Ralph will work on:**
US-XXX: [title]

**To run Ralph:**
./scripts/ralph/ralph.sh [max_iterations]
```

---

## Acceptance Criteria Requirements

Each story MUST include verifiable acceptance criteria:

### Good criteria (verifiable):
- "Add `status` column to tasks table with default 'pending'"
- "Filter dropdown has options: All, Active, Completed"
- "Typecheck passes"
- "Tests pass"

### Bad criteria (vague):
- ❌ "Works correctly"
- ❌ "Good UX"

### Always include as final criterion:
```
"Typecheck passes"
```

### For UI stories:
```
"Verify in browser using dev-browser skill"
```

---

## Example: Mode 1 Conversation

**User:** set up ralph for adding a priority field to tasks

**Assistant:**

First, let me ensure the ralph files exist...
[Creates ralph.sh and prompt.md if needed]

What feature are you building?

**User:** I want to add priority levels (low, medium, high) to tasks.

**Assistant:** Got it. A few questions:
1. Should priority be required or optional?
2. Where should the selector appear?
3. Should the list auto-sort by priority?

**User:** Optional with default "medium". Both create and edit. Yes, sort by priority.

**Assistant:** Here's my breakdown:

1. **US-001: Add priority column** - Schema change
2. **US-002: Display priority badge** - Visual indicator
3. **US-003: Add priority selector** - Dropdown in forms
4. **US-004: Sort by priority** - Update query
5. **US-005: Filter by priority** - Add filter

Does this look right?

**User:** Yes, create it

**Assistant:** [Creates prd.json, progress.txt]

✅ Ralph is ready!
...

---

## Example: Mode 3 Conversation

**User:** I want to build a habit tracker app

**Assistant:**

First, let me ensure the ralph files exist...
[Creates ralph.sh and prompt.md if needed]

Tell me about the habit tracker:
- What's the core goal?
- What features do you want?
- Any technical constraints?

**User:** Users create habits, mark them complete daily, see streaks. React + Node.

**Assistant:** Let me understand the scope:

1. **Habits CRUD** - create, edit, delete
2. **Daily tracking** - mark complete
3. **Streaks** - current and best
4. **Dashboard** - overview

Anything else? Auth?

**User:** No auth for MVP. Maybe a stats page.

**Assistant:** Here's the full breakdown:

**Feature 1: Database (4 stories)**
- US-001: habits table
- US-002: completions table
- US-003: streak calculation
- US-004: stats helpers

**Feature 2: API (4 stories)**
- US-005: CRUD endpoints
- US-006: mark complete
- US-007: get today's habits
- US-008: get stats

**Feature 3: UI (4 stories)**
- US-009: habit list
- US-010: add/edit modal
- US-011: check-off UI
- US-012: streak display

**Feature 4: Dashboard (3 stories)**
- US-013: today's habits
- US-014: stats page
- US-015: progress indicators

Total: 15 stories. Create prd.json?

**User:** Yes

**Assistant:** [Creates prd.json with all stories]

✅ Product plan created!
...

---

## Checklist Before Creating prd.json

- [ ] Ralph files exist (ralph.sh, prompt.md)
- [ ] Chatted through feature(s) to understand scope
- [ ] Each story completable in one iteration
- [ ] Stories ordered by dependency
- [ ] Every story has "Typecheck passes"
- [ ] UI stories have browser verification
- [ ] Acceptance criteria are verifiable

---

## Running Ralph

After setup is complete:

```bash
./scripts/ralph/ralph.sh 20
```

Ralph will:
1. Read prd.json, find next `passes: false` story
2. Implement it
3. Run quality checks
4. Commit if passing
5. Set `passes: true`
6. Log to progress.txt
7. Repeat until all done or max iterations

When all stories pass, Ralph outputs `<promise>COMPLETE</promise>` and exits.
