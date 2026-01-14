---
name: ralph
description: "Set up Ralph for autonomous feature development. Use when starting a new feature that Ralph will implement. Triggers on: ralph, set up ralph, start ralph, new ralph feature, ralph setup. Chats through the feature idea, creates tasks with dependencies, and sets up everything for Ralph to run."
---

# Ralph Feature Setup

Interactive feature planning that creates ralph-ready tasks with dependencies.

---

## The Job

**Three modes:**

### Mode 1: New Feature
1. Chat through the feature - Ask clarifying questions
2. Break into small tasks - Each completable in one iteration
3. Create task_list tasks - Parent + subtasks with `dependsOn`
4. Set up ralph files - Save parent ID, reset progress.txt

### Mode 2: Existing Tasks
1. Find existing parent task - Search or let user specify
2. Verify structure - Check subtasks have proper `dependsOn`
3. Set up ralph files - Save parent ID to parent-task-id.txt
4. Show status - Which tasks are ready, completed, blocked

### Mode 3: Full Product Build
1. Chat through the product vision - Understand the complete scope
2. Break into major features/epics - Each feature becomes a parent task
3. Break each feature into small tasks - Same sizing rules as Mode 1
4. Create hierarchical task structure - Product → Features → Tasks
5. Set up ralph files - Start with first feature's parent ID
6. Plan the feature sequence - Which features to build first

**Ask the user which mode they need:**
```
Are you:
1. Starting a new feature (I'll help you plan and create tasks)
2. Using existing tasks (I'll set up Ralph to run them)
3. Building a full product from scratch (I'll help you plan the entire product)
```

---

## Step 1: Understand the Feature

Start by asking the user about their feature. Don't assume - ASK:

```
What feature are you building?
```

Then ask clarifying questions:
- What's the user-facing goal?
- What parts of the codebase will this touch? (database, UI, API, etc.)
- Are there any existing patterns to follow?
- What should it look like when done?

**Keep asking until you have enough detail to break it into tasks.**

---

## Step 2: Break Into Tasks

**Each task must be completable in ONE Ralph iteration (~one context window).**

Ralph spawns a fresh Claude instance per iteration with no memory of previous work. If a task is too big, the LLM runs out of context before finishing.

### Right-sized tasks:
- Add a database column + migration
- Create a single UI component
- Implement one server action
- Add a filter to an existing list
- Write tests for one module

### Too big (split these):
- "Build the entire dashboard" → Split into: schema, queries, components, filters
- "Add authentication" → Split into: schema, middleware, login UI, session handling
- "Refactor the API" → Split into one task per endpoint

**Rule of thumb:** If you can't describe the change in 2-3 sentences, it's too big.

---

## Step 3: Order by Dependencies

Tasks execute based on `dependsOn`. Earlier tasks must complete before dependent ones start.

**Typical order:**
1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Integration / E2E tests

Use `dependsOn` to express this:
```
Task 1: Schema (no dependencies)
Task 2: Server action (dependsOn: [task-1])
Task 3: UI component (dependsOn: [task-2])
Task 4: Tests (dependsOn: [task-3])
```

Parallel tasks that don't depend on each other can share the same dependency.

---

## Step 4: Create Tasks

### First, create the parent task:

```
task_list create
  title: "[Feature Name]"
  description: "[One-line description of the feature]"
  repoURL: "https://github.com/snarktank/untangle"
```

**Save the returned task ID** - you'll need it for subtasks.

### Then, create subtasks with parentID and dependsOn:

```
task_list create
  title: "[Task title - action-oriented]"
  description: "[Detailed description with:
    - What to implement
    - Files to create/modify
    - Acceptance criteria
    - How to verify (typecheck, tests, browser)]"
  parentID: "<parent-task-id>"
  dependsOn: ["<previous-task-id>"]  // omit for first task
  repoURL: "https://github.com/snarktank/untangle"
```

### Task description format:

Write descriptions that a future Ralph iteration can pick up without context:

```
Implement category name to ID mapping for expenses.

**What to do:**
- Create function mapExpenseCategoryNameToId(name, isChildExpense)
- Query item_category table with category_type filter
- Add alias mapping for common synonyms (rent → Rent or Mortgage)

**Files:**
- workflows/tools/upsert-expense.ts

**Acceptance criteria:**
- Function returns category ID for valid names
- Returns null for unknown categories
- npm run typecheck passes

**Notes:**
- Follow pattern from upsert-income.ts
- EXPENSE type for family, CHILD_EXPENSE for child
```

---

## Step 5: Set Up Ralph Files

After creating all tasks, **run the shared setup steps from "Final Setup (Required for Both Modes)" section.**

This ensures:
- Parent task ID is saved to `scripts/ralph/parent-task-id.txt`
- Previous progress.txt is archived if it has content
- Fresh progress.txt is created with Codebase Patterns preserved

---

## Step 6: Confirm Setup

Show the user what was created:

```
✅ Ralph is ready!

**Parent task:** [title] (ID: [id])

**Subtasks:**
1. [Task 1 title] - no dependencies
2. [Task 2 title] - depends on #1
3. [Task 3 title] - depends on #2
...

**To run Ralph:**
```bash
./scripts/ralph/ralph.sh [max_iterations]
# or directly:
npx tsx scripts/ralph/ralph.ts [max_iterations]
```

**To check status:**
```bash
# Use your task management tool to list subtasks under [parent-id]
```

---

## Mode 2: Setting Up Existing Tasks

If the user already has tasks created, help them set up Ralph to run them.

### Find the parent task:

Ask the user:
```
What's the parent task? You can give me:
- The task ID directly
- A search term and I'll find it
- Or say "list recent" to see recent tasks
```

To search for tasks (always use limit to avoid context overflow):
```
task_list list
  repoURL: "https://github.com/snarktank/untangle"
  limit: 10
```

### Verify subtasks exist:

Once you have the parent ID, check for subtasks (always use limit):
```
task_list list
  parentID: "<parent-task-id>"
  limit: 10
```

If no subtasks found, the parent task might BE the work (not a container). Ask:
```
This task has no subtasks. Is this:
1. A parent task with subtasks I should find differently?
2. The actual work task (I should create it as a parent with this as the first subtask)?
```

### Check dependencies:

Review the subtasks and verify:
- Do they have `dependsOn` set correctly?
- Are there any circular dependencies?
- Is the first task(s) dependency-free so Ralph can start?

If dependencies are missing, offer to fix:
```
These tasks don't have dependencies set. Should I:
1. Add dependencies based on their order?
2. Leave them parallel (Ralph picks any ready task)?
```

### Set up ralph files:

**Run the shared setup steps from "Final Setup (Required for Both Modes)" section below.**

### Show status:

```
✅ Ralph is ready to use existing tasks!

**Parent task:** [title] (ID: [id])

**Status:**
- ✅ Completed: 3 tasks
- 🔄 Ready to work: 2 tasks
- ⏳ Blocked: 5 tasks (waiting on dependencies)

**Next task Ralph will pick:**
[Task title] - [brief description]

**To run Ralph:**
```bash
./scripts/ralph/ralph.sh [max_iterations]
# or directly:
npx tsx scripts/ralph/ralph.ts [max_iterations]
```

---

## Mode 3: Building a Full Product

For building an entire product from scratch, we need a hierarchical approach.

### Step 1: Understand the Product Vision

Start with big-picture questions:
```
What product are you building? Tell me about:
- What problem does it solve?
- Who is the target user?
- What are the core features (the MVP)?
- What's the tech stack? (or should I recommend one?)
```

Then dig deeper:
- What does the user journey look like?
- Are there any third-party integrations needed?
- What's the authentication/authorization model?
- Is there a database? What entities need to be stored?

**Keep asking until you understand the full scope.**

### Step 2: Break Into Features/Epics

Identify the major features that make up the product. Each feature will become its own parent task with subtasks.

**Typical product breakdown:**
1. **Project Setup** - Initialize repo, install dependencies, configure tooling
2. **Database/Schema** - Design and create the data model
3. **Authentication** - User signup, login, sessions
4. **Core Feature 1** - The main value proposition
5. **Core Feature 2** - Secondary functionality
6. **UI/Layout** - Navigation, layout, styling
7. **Integration** - Third-party services, APIs
8. **Polish** - Error handling, loading states, edge cases
9. **Testing** - Unit tests, integration tests, E2E
10. **Deployment** - CI/CD, hosting, environment setup

### Step 3: Order Features by Dependencies

Features have dependencies just like tasks:

```
1. Project Setup (no dependencies)
2. Database/Schema (depends on: setup)
3. Authentication (depends on: schema)
4. Core Features (depends on: auth, schema)
5. UI/Layout (depends on: core features)
6. Testing (depends on: all features)
7. Deployment (depends on: testing)
```

### Step 4: Create the Task Hierarchy

**Level 1: Product (top-level parent)**
```
task_list create
  title: "[Product Name]"
  description: "Full product build: [one-line description]"
  repoURL: "<repo-url>"
```

**Level 2: Features (children of product)**
```
task_list create
  title: "[Feature Name]"
  description: "[Feature description and scope]"
  parentID: "<product-task-id>"
  dependsOn: ["<previous-feature-id>"]  // if sequential
  repoURL: "<repo-url>"
```

**Level 3: Tasks (children of features)**
```
task_list create
  title: "[Task title]"
  description: "[Detailed task description]"
  parentID: "<feature-task-id>"
  dependsOn: ["<previous-task-id>"]
  repoURL: "<repo-url>"
```

### Step 5: Set Up for First Feature

Ralph works on one feature at a time. Set up for the first feature:

1. Save the **first feature's ID** (not the product ID) to parent-task-id.txt
2. Ralph will complete all tasks in that feature
3. When done, update parent-task-id.txt to the next feature
4. Repeat until all features are complete

### Step 6: Confirm Product Setup

Show the user the full plan:

```
✅ Product plan created!

**Product:** [name] (ID: [product-id])

**Features (in order):**
1. [Feature 1] (ID: [id]) - [X] tasks - no dependencies
2. [Feature 2] (ID: [id]) - [X] tasks - depends on #1
3. [Feature 3] (ID: [id]) - [X] tasks - depends on #2
...

**Total:** [N] features, [M] tasks

**First feature to build:** [Feature 1]
- Task 1: [title]
- Task 2: [title]
- ...

**To start Ralph on Feature 1:**
```bash
./scripts/ralph/ralph.sh [max_iterations]
```

**When Feature 1 is complete, update to Feature 2:**
```bash
echo "<feature-2-id>" > scripts/ralph/parent-task-id.txt
./scripts/ralph/ralph.sh [max_iterations]
```
```

### Product Build Tips

**Start small:** Begin with the minimal viable feature set. You can always add more features later.

**Vertical slices:** Each feature should be a complete vertical slice (database → backend → frontend) so you have working functionality after each feature.

**Test as you go:** Include testing tasks within each feature, not as a separate feature at the end.

**Keep features independent:** Where possible, design features to be buildable in any order. This gives flexibility.

### Example Product Breakdown

**Product:** Personal Finance Tracker

**Features:**
1. **Project Setup** (3 tasks)
   - Initialize Next.js project with TypeScript
   - Set up database (Postgres + Prisma)
   - Configure authentication (NextAuth)

2. **Transaction Management** (8 tasks)
   - Create transaction schema
   - Build transaction list UI
   - Add transaction form
   - Implement CRUD operations
   - Add category support
   - Implement filtering
   - Add pagination
   - Write tests

3. **Dashboard** (5 tasks)
   - Create dashboard layout
   - Build spending summary component
   - Add charts/visualizations
   - Implement date range picker
   - Write tests

4. **Budget Tracking** (6 tasks)
   - Create budget schema
   - Build budget list UI
   - Add budget form
   - Implement budget vs actual comparison
   - Add alerts for overspending
   - Write tests

5. **Deployment** (3 tasks)
   - Set up Vercel deployment
   - Configure environment variables
   - Add CI/CD pipeline

---

## Final Setup (Required for All Modes)

**ALWAYS run these steps after creating tasks OR setting up existing tasks:**

### 1. Save parent task ID:

```bash
echo "<parent-task-id>" > scripts/ralph/parent-task-id.txt
```

Verify it was saved:
```bash
cat scripts/ralph/parent-task-id.txt
```

### 2. Check if progress.txt needs archiving:

Read the current progress.txt:
```bash
cat scripts/ralph/progress.txt
```

**Archive if:**
- It has content beyond the header (learnings from a previous feature)
- The previous feature is different from the current one

**Archive command:**
```bash
DATE=$(date +%Y-%m-%d)
FEATURE="previous-feature-name-here"
mkdir -p scripts/ralph/archive/$DATE-$FEATURE
cp scripts/ralph/progress.txt scripts/ralph/archive/$DATE-$FEATURE/
echo "Archived to scripts/ralph/archive/$DATE-$FEATURE/"
```

### 3. Reset progress.txt:

**Preserve useful Codebase Patterns** from the previous run, then reset:

```bash
cat > scripts/ralph/progress.txt << 'EOF'
# Ralph Progress Log
Started: [current date]

## Codebase Patterns
[Copy any patterns from previous run that are still relevant]
---
EOF
```

### 4. Verify setup is complete:

```bash
# Confirm parent ID is saved
cat scripts/ralph/parent-task-id.txt

# Confirm progress.txt is reset
head -10 scripts/ralph/progress.txt

# List subtasks to confirm they exist
# (use task_list list with parentID)
```

**Only after completing all 4 steps is Ralph ready to run.**

---

## Acceptance Criteria for Each Task

Each task description MUST include verifiable acceptance criteria:

### Good criteria (verifiable):
- "Add `investorType` column to investor table with default 'cold'"
- "Filter dropdown has options: All, Cold, Friend"
- "npm run typecheck passes"
- "npm test -- path/to/test.ts passes"

### Bad criteria (vague):
- ❌ "Works correctly"
- ❌ "Good UX"
- ❌ "Handles edge cases"

### Always include:
```
npm run typecheck passes
```

### For tasks with testable logic:
```
npm test passes
```

### For UI/browser tasks:

**For functional testing** (checking behavior, not appearance):
```
Use Chrome DevTools MCP with take_snapshot to read page content (NOT screenshots)
```
- `take_snapshot` returns the a11y tree as text that Ralph can read and verify
- `take_screenshot` returns images which Ralph cannot interpret for functional checks
- Only use screenshots when testing visual appearance/look and feel

**For visual testing** (checking appearance):
```
Use take_screenshot to capture and verify visual appearance
```

---

## Example Conversation

**User:** set up ralph for adding expense tracking to the legal agent

**Assistant:** What feature are you building?

**User:** I want Grace (our legal chat agent) to be able to create, update, and delete expenses when users mention them in conversation.

**Assistant:** Got it - an expense upsert tool for the legal agent. A few questions:

1. Are there existing tools in the legal agent I should follow as a pattern?
2. Do expenses have different types (family vs child)?
3. Should it also list expenses or just CRUD?
4. Any task completion logic needed?

**User:** Follow upsert-income.ts. Yes family and child expenses (different tables). Yes list too. Mark APP_EXPENSES_ENTERED task complete.

**Assistant:** Perfect. Here's my proposed breakdown:

1. **Create tool skeleton** - Zod schema, basic structure
2. **Category mapping** - Map names like "rent" to category IDs
3. **Amount parsing** - Handle currency formats, allow $0
4. **Duplicate detection** - Check before creating
5. **Create family expenses** - Insert into expense table
6. **Create child expenses** - Insert into child_expense table
7. **Update action** - Modify existing expenses
8. **Delete action** - Remove expenses
9. **List action** - Show all expenses with totals
10. **Task completion** - Mark APP_EXPENSES_ENTERED done
11. **Register tool** - Add to legal chat workflow
12. **UI progress indicator** - Show progress during tool use
13. **Unit tests** - Test helper functions
14. **E2E tests** - Test via chat interface

Does this look right? Any tasks to add or remove?

**User:** Looks good, create the tasks

**Assistant:** [Creates parent task, then subtasks with dependencies, saves parent-task-id.txt]

✅ Ralph is ready!

**Parent task:** Legal Agent Expense Upsert Tool (ID: task-abc123)

**Subtasks:** 14 tasks created with dependencies

**To run:** `./scripts/ralph/ralph.sh 20`

---

## How Ralph Completes

When all subtasks are completed:
1. Ralph marks the **parent task** as `completed`
2. Ralph outputs `<promise>COMPLETE</promise>`
3. The ralph.sh/ralph.ts loop detects this and exits

**Important:** Ralph uses `limit: 5` when querying tasks to avoid context overflow. If you have many subtasks, they'll be processed over multiple iterations.

---

## Checklist Before Creating Tasks

**For all modes:**
- [ ] Each task completable in one iteration (small enough)
- [ ] Tasks ordered by dependency (schema → backend → UI → tests)
- [ ] Every task has "npm run typecheck passes" in description
- [ ] UI tasks have browser verification in description
- [ ] Descriptions have enough detail for Ralph to implement without context
- [ ] Parent task ID saved to scripts/ralph/parent-task-id.txt
- [ ] Previous run archived if progress.txt had content

**For Mode 1 (New Feature):**
- [ ] Chatted through feature to understand scope

**For Mode 2 (Existing Tasks):**
- [ ] Verified subtasks exist with proper dependencies
- [ ] Confirmed at least one task is ready (no blockers)

**For Mode 3 (Full Product):**
- [ ] Understood complete product vision and scope
- [ ] Broke product into distinct features/epics
- [ ] Created hierarchical structure (Product → Features → Tasks)
- [ ] Features ordered by dependencies
- [ ] First feature's ID saved to parent-task-id.txt (not product ID)
- [ ] Documented feature sequence for manual progression
