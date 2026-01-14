---
name: ralph
description: "Autonomous feature development - setup and execution. Triggers on: ralph, set up ralph, run ralph, run the loop, implement tasks. Three modes: (1) New Feature - plan and create tasks for a single feature (2) Existing Tasks - set up Ralph for existing prd.json (3) Full Product - plan and create tasks for entire product with multiple features."
---

# Ralph Feature Setup

Interactive feature planning that creates ralph-ready tasks in prd.json format.

---

## The Job

**Three modes:**

### Mode 1: New Feature (Single Feature)
1. Chat through the feature - Ask clarifying questions
2. Break into small user stories - Each completable in one iteration
3. Create prd.json - Stories with `passes: false`
4. Set up ralph files - Reset progress.txt

### Mode 2: Existing Tasks
1. Verify prd.json exists and is valid
2. Show current status - Which stories pass/fail
3. Set up ralph files if needed

### Mode 3: Full Product Build (Multiple Features)
1. Chat through the product - Ask about features, scope, requirements
2. Break ALL features into small user stories - Same sizing rules as Mode 1
3. Create prd.json with all stories - Ordered by dependency (schema → backend → UI)
4. Set up ralph files - Same as Mode 1
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

### Output Format:

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

### Story description format:

Write descriptions that a future Ralph iteration can pick up without context:

```
As a developer, I need to store task status in the database.

**What to do:**
- Add status column to tasks table
- Create migration with default value 'pending'
- Update schema types

**Files:**
- db/schema.ts
- db/migrations/

**Notes:**
- Follow existing migration pattern
```

---

## Step 5: Set Up Ralph Files

After creating prd.json:

### 1. Check if previous run needs archiving:

```bash
cat prd.json 2>/dev/null | jq -r '.branchName'
```

If prd.json exists with a DIFFERENT branchName and progress.txt has content:
```bash
DATE=$(date +%Y-%m-%d)
FEATURE="previous-feature-name"
mkdir -p archive/$DATE-$FEATURE
cp prd.json archive/$DATE-$FEATURE/
cp progress.txt archive/$DATE-$FEATURE/
```

### 2. Write prd.json:

Save the new prd.json to the ralph directory (or project root).

### 3. Reset progress.txt:

```bash
cat > progress.txt << 'EOF'
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

### For Mode 1 (Single Feature):

```
✅ Ralph is ready!

**Feature:** [name]
**Branch:** ralph/[feature-name]

**User Stories:**
1. US-001: [title] - priority 1
2. US-002: [title] - priority 2
3. US-003: [title] - priority 3
...

**To run Ralph:**
./ralph.sh [max_iterations]

Ralph will work through each story in priority order until all pass.
```

### For Mode 3 (Full Product):

```
✅ Product plan created!

**Product:** [name]
**Branch:** ralph/[product-name]

**Stories by feature:**

Feature 1: [name]
- US-001: [title] - priority 1
- US-002: [title] - priority 2

Feature 2: [name]
- US-003: [title] - priority 3
- US-004: [title] - priority 4

...

**Total:** [N] stories across [X] features

**To run Ralph:**
./ralph.sh [max_iterations]

Ralph will work through ALL stories until the entire product is complete.
```

---

## Mode 2: Setting Up Existing Tasks

If the user already has prd.json:

### 1. Verify prd.json exists and is valid:

```bash
cat prd.json | jq '.userStories | length'
```

### 2. Show current status:

```
**Current Status:**
- ✅ Passing: [N] stories
- ❌ Not passing: [M] stories

**Next story Ralph will work on:**
US-XXX: [title]

**To run Ralph:**
./ralph.sh [max_iterations]
```

### 3. Fix issues if needed:

If stories are missing acceptance criteria or are too big, offer to fix:
```
Some stories might need adjustment:
- US-003 looks too big - want me to split it?
- US-005 is missing "Typecheck passes" criterion - want me to add it?
```

---

## Acceptance Criteria Requirements

Each story MUST include verifiable acceptance criteria:

### Good criteria (verifiable):
- "Add `status` column to tasks table with default 'pending'"
- "Filter dropdown has options: All, Active, Completed"
- "Clicking delete shows confirmation dialog"
- "Typecheck passes"
- "Tests pass"

### Bad criteria (vague):
- ❌ "Works correctly"
- ❌ "Good UX"
- ❌ "Handles edge cases"

### Always include as final criterion:
```
"Typecheck passes"
```

### For stories with testable logic:
```
"Tests pass"
```

### For UI stories:
```
"Verify in browser using dev-browser skill"
```

---

## Example: Mode 1 Conversation

**User:** set up ralph for adding a priority field to tasks

**Assistant:** What feature are you building?

**User:** I want to add priority levels (low, medium, high) to tasks so users can sort and filter by importance.

**Assistant:** Got it - task priority feature. A few questions:

1. Should priority be required or optional (with a default)?
2. Where should the priority selector appear - task creation, edit, or both?
3. Should the task list auto-sort by priority?

**User:** Optional with default "medium". Show in both create and edit. Yes, sort by priority then by date.

**Assistant:** Here's my proposed breakdown:

1. **US-001: Add priority column** - Schema change with migration
2. **US-002: Display priority badge** - Visual indicator on task cards
3. **US-003: Add priority selector** - Dropdown in create/edit forms
4. **US-004: Sort by priority** - Update list query and UI
5. **US-005: Filter by priority** - Add filter dropdown

Does this look right?

**User:** Looks good, create the tasks

**Assistant:** [Creates prd.json, resets progress.txt]

✅ Ralph is ready!
...

---

## Example: Mode 3 Conversation

**User:** I want to build a simple habit tracker app

**Assistant:** Tell me about the habit tracker:
- What's the core goal?
- What features do you want?
- Any technical constraints?

**User:** Users can create habits, mark them complete daily, and see streaks. Web app, React + Node.

**Assistant:** Got it. Let me understand the scope:

1. **Habits CRUD** - create, edit, delete habits
2. **Daily tracking** - mark habits complete for today
3. **Streaks** - show current streak and best streak
4. **Dashboard** - overview of today's habits and progress

Anything else? Authentication? Multiple users?

**User:** No auth for MVP, just local. Maybe a simple stats page showing completion rate.

**Assistant:** Here's the full breakdown:

**Feature 1: Database & Schema (4 stories)**
- US-001: Set up database with habits table
- US-002: Add completions table (habit_id, date)
- US-003: Add streak calculation logic
- US-004: Add stats calculation helpers

**Feature 2: Backend API (4 stories)**
- US-005: Habits CRUD endpoints
- US-006: Mark habit complete endpoint
- US-007: Get today's habits with status
- US-008: Get stats endpoint

**Feature 3: Core UI (4 stories)**
- US-009: Habit list component
- US-010: Add/edit habit modal
- US-011: Daily check-off UI
- US-012: Streak display component

**Feature 4: Dashboard & Stats (3 stories)**
- US-013: Today's habits dashboard
- US-014: Stats page with completion rate
- US-015: Visual progress indicators

Total: 15 stories. Should I create the prd.json?

**User:** Yes, create it

**Assistant:** [Creates prd.json with all 15 stories, properly ordered]

✅ Product plan created!
...

---

## Checklist Before Creating prd.json

- [ ] Chatted through feature(s) to understand scope
- [ ] Each story completable in one iteration (small enough)
- [ ] Stories ordered by dependency (schema → backend → UI)
- [ ] Every story has "Typecheck passes" as criterion
- [ ] UI stories have "Verify in browser using dev-browser skill" as criterion
- [ ] Acceptance criteria are specific and verifiable
- [ ] No story depends on a later story
- [ ] Previous run archived if prd.json exists with different branchName

---

# Phase 2: The Execution Loop

Once setup is complete, Ralph runs the autonomous loop to implement stories one by one.

---

## Loop Workflow

### 0. Read prd.json

```bash
cat prd.json
```

Find the first story where `passes: false`, ordered by priority.

### 1. If no incomplete stories

All stories have `passes: true`:
1. Output `<promise>COMPLETE</promise>`
2. Archive progress.txt
3. Report "✅ Build complete - all stories finished!"

### 2. If incomplete stories exist

**Pick the next story:**
- Find lowest priority number where `passes: false`
- This ensures dependency order is respected

**Execute the story:**

Implement the acceptance criteria, then:

1. Run quality checks: `npm run typecheck` (and tests if applicable)
   - If fails, FIX THE ISSUES and re-run until passing
   - Do NOT proceed until quality checks pass

2. Update AGENTS.md if you learned something important:
   - Add patterns, gotchas, conventions for future developers/iterations

3. Update progress.txt (APPEND):
   ```
   ## [Date] - US-XXX: [Story Title]
   - What was implemented
   - Files changed
   - Learnings for future iterations
   ---
   ```

4. Commit changes: `git add . && git commit -m "feat: [Story Title]"`

5. Update prd.json: Set `passes: true` for this story, add notes if relevant

6. Continue to next story (or exit if all complete)

---

## Progress File Format

```markdown
# Ralph Progress Log
Started: [date]
Feature: [feature/product name]

## Codebase Patterns
(Patterns discovered during this build - useful for remaining stories)

---

## [Date] - US-001: [Story Title]
- What was implemented
- Files changed
- Learnings for future iterations
---

## [Date] - US-002: [Story Title]
- What was implemented
- Files changed
---
```

---

## Browser Verification

For UI stories, use the dev-browser skill:

**Functional testing** (checking behavior):
- Navigate to the page
- Interact with elements
- Verify expected behavior

**Visual testing** (checking appearance):
- Take screenshots
- Verify layout and styling

---

## Quality Requirements

Before marking any story as `passes: true`:
- Typecheck must pass
- Tests must pass (if applicable)
- Changes must be committed
- Progress must be logged

---

## Stop Condition

When all stories have `passes: true`:
1. Output: `<promise>COMPLETE</promise>`
2. Ralph.sh detects this and exits the loop
3. Feature/product is complete!

---

## Important Notes

- Each iteration runs in a fresh Claude instance with clean context
- prd.json is the source of truth for what's done
- progress.txt provides context between iterations
- Stories execute in priority order - respect the dependencies
- Keep stories small enough to complete in one context window
