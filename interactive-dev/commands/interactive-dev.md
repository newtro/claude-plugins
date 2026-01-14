---
name: interactive-dev
description: Start an interactive development workflow with planning, implementation, and verification loops
---

# Interactive Development Workflow

You are orchestrating an interactive development workflow. This command runs in the main agent so you can use `AskUserQuestion` for multi-choice requirement gathering.

## Workflow Overview

1. **Planning Phase**: Interview user to gather requirements
2. **Criteria Phase**: Define testable done criteria with user
3. **Implementation Loop**: Code → Verify → Fix until all criteria pass

## Phase 1: Planning

### Load the Planning Skill

Reference the planning skill guidance from `skills/planning/SKILL.md` in this plugin.

### Interview the User

Use `AskUserQuestion` to gather requirements interactively. Structure your questions following the planning skill's guidance:

**Question Format:**
```
{
  "questions": [
    {
      "question": "What should happen when the user submits the form?",
      "header": "Submit",
      "options": [
        {"label": "Show success message (Recommended)", "description": "Display a confirmation message on the same page"},
        {"label": "Redirect to dashboard", "description": "Navigate to the main dashboard after success"},
        {"label": "Show confirmation modal", "description": "Display a modal dialog with confirmation"}
      ],
      "multiSelect": false
    }
  ]
}
```

**Interview Flow:**
1. Start with understanding the core feature (1-2 questions)
2. Define scope and boundaries (2-3 questions)
3. Clarify UI/UX details (2-3 questions)
4. Address edge cases if complex (1-2 questions)

Keep the interview focused - typically 5-8 questions total.

## Phase 2: Define Done Criteria

### Load the Done Criteria Skill

Reference the done-criteria skill guidance from `skills/done-criteria/SKILL.md` in this plugin.

### Create Criteria with User

Based on the requirements gathered, propose done criteria to the user. Use `AskUserQuestion` to confirm:

```
{
  "questions": [
    {
      "question": "Which of these criteria should be included for this feature?",
      "header": "Criteria",
      "options": [
        {"label": "Form displays email and password fields", "description": "Basic form structure"},
        {"label": "Submit shows loading state", "description": "UX feedback during submission"},
        {"label": "Invalid inputs show error messages", "description": "Validation feedback"},
        {"label": "Successful login redirects to dashboard", "description": "Success flow"}
      ],
      "multiSelect": true
    }
  ]
}
```

### Always Include Build Criteria

Add these standard criteria:
- Build completes without errors
- No TypeScript/type errors (if applicable)
- No lint errors (if applicable)

## Phase 3: Create Task File

### Generate Task File

Create a timestamped task file in `.claude/tasks/`:

**Filename format:** `YYYY-MM-DD-HHMM-[slug].md`

Example: `.claude/tasks/2024-01-15-1430-add-login-form.md`

**Content format:**
```markdown
# Task: [Title]

## Requirements
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

## Done Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]
- [ ] Build completes without errors

## Technical Decisions
- [Decision 1 from interview]
- [Decision 2 from interview]

## Files Changed
(Updated by coding agent)
```

### Update Task Index

If `.claude/tasks/index.md` exists, add an entry. If not, create it:

```markdown
# Task Index

| Date | Task | Status |
|------|------|--------|
| 2024-01-15 | [Add Login Form](./2024-01-15-1430-add-login-form.md) | In Progress |
```

## Phase 4: Implementation Loop

### Loop Structure

```
max_iterations = 5
iteration = 0

while iteration < max_iterations:
    iteration += 1

    # Step 1: Invoke coding agent
    coding_result = Task(
        subagent_type="coding-agent",
        prompt=f"Implement the feature described in {task_file_path}. Read the requirements and done criteria, write the code, and run build/lint/typecheck until clean."
    )

    # Step 2: Read updated task file for changed files
    changed_files = read_files_changed_section(task_file_path)

    # Step 3: Invoke verification agent
    verification_result = Task(
        subagent_type="verification-agent",
        prompt=f"Verify the implementation in {task_file_path}. Changed files: {changed_files}. Test each criterion using Playwright MCP and update checkboxes."
    )

    # Step 4: Check completion
    criteria = read_done_criteria(task_file_path)

    if all_criteria_checked(criteria):
        # Success! Mark task complete
        update_task_status("Complete")
        notify_user("All criteria passed!")
        break
    else:
        # Get failure details
        failures = get_unchecked_criteria(criteria)
        notify_user(f"Iteration {iteration}: {len(failures)} criteria still failing")

        if iteration < max_iterations:
            # Pass failures back to coding agent in next iteration
            continue
        else:
            notify_user("Max iterations reached. Review task file for remaining issues.")
```

### Invoking Agents

Use the `Task` tool to invoke subagents:

**Coding Agent:**
```
Task(
    subagent_type="coding-agent",
    prompt="Implement the feature in .claude/tasks/2024-01-15-1430-add-login-form.md. Previous issues to address: [list any failures from verification]"
)
```

**Verification Agent:**
```
Task(
    subagent_type="verification-agent",
    prompt="Verify .claude/tasks/2024-01-15-1430-add-login-form.md. Changed files: LoginForm.tsx, login.tsx, useAuth.ts"
)
```

### Handling Failures

After each verification:
1. Read the updated task file
2. Identify unchecked criteria
3. Extract failure reasons from verification agent's response
4. Pass specific issues to the coding agent in the next iteration

Example prompt for retry:
```
"Continue implementing .claude/tasks/2024-01-15-1430-add-login-form.md.

The following criteria are still failing:
- [ ] Invalid login shows error message - Verification found: No error message appears after invalid credentials
- [ ] Form validation prevents empty submission - Verification found: Form submits with empty fields

Focus on fixing these specific issues."
```

## Phase 5: Completion

### When All Criteria Pass

1. Update task index status to "Complete"
2. Summarize to user:
   - What was built
   - Files that were changed
   - Number of iterations needed

### When Max Iterations Reached

1. Update task index status to "Incomplete"
2. Report to user:
   - Which criteria passed
   - Which criteria still fail
   - Suggest manual review

## User Argument Handling

The command receives the user's task description as `$ARGUMENTS`.

If `$ARGUMENTS` is provided:
- Use it as the starting point for the interview
- Begin with clarifying questions about the description

If `$ARGUMENTS` is empty:
- Ask the user what they want to build
- Start the interview from scratch

## Example Session

```
User: /interactive-dev Add a user profile page

Agent: I'll help you build a user profile page. Let me ask a few questions to understand the requirements.

[AskUserQuestion: What information should the profile display?]
User selects: Name, email, avatar, bio

[AskUserQuestion: Should users be able to edit their profile?]
User selects: Yes, inline editing

[AskUserQuestion: How should the page handle loading?]
User selects: Show skeleton loader

Agent: Based on your answers, here are the proposed done criteria...

[AskUserQuestion: Confirm criteria]
User confirms selection

Agent: Creating task file and starting implementation...

[Task: coding-agent implements feature]
[Task: verification-agent tests feature]

Agent: Iteration 1 complete. 4/5 criteria passing. Running another iteration...

[Task: coding-agent fixes remaining issue]
[Task: verification-agent verifies fix]

Agent: All criteria passing! Your user profile page is complete.

Files changed:
- src/components/UserProfile.tsx
- src/pages/profile.tsx
- src/hooks/useProfile.ts
```
