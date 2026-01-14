---
name: done-criteria
description: Helps define testable acceptance criteria for development tasks. Use when establishing done criteria during interactive-dev planning.
---

# Done Criteria Skill

This skill guides you in defining clear, testable acceptance criteria that can be verified by automated testing and browser-based verification.

## What Makes Good Done Criteria

Every criterion must be:

### 1. Observable
Can be seen in the browser or test output. Something a human or automated test can verify.

### 2. Specific
Not vague or subjective. Clear enough that two different testers would agree on pass/fail.

### 3. Testable
Can be verified programmatically or through browser interaction. No "feel" or "seem" language.

## Categories of Criteria

### Functional Criteria
The feature works as expected.

Examples:
- "Clicking 'Submit' button sends form data to the server"
- "User can navigate between pages using the sidebar menu"
- "Search results update as user types in the search box"

### Visual Criteria
The UI renders correctly.

Examples:
- "Profile page displays user name, email, and avatar image"
- "Error messages appear in red text below the input field"
- "Loading spinner is visible while data is fetching"

### Validation Criteria
Errors and edge cases are handled properly.

Examples:
- "Form shows 'Email is required' when submitted without email"
- "Empty state message appears when no results are found"
- "Network error displays 'Unable to connect' message"

### Build Criteria
Code quality standards are met.

Examples:
- "Build completes without errors"
- "No TypeScript type errors"
- "No ESLint warnings"
- "All existing tests pass"

## Examples: Good vs Bad Criteria

### Bad Criteria (Avoid These)

| Bad | Problem |
|-----|---------|
| "Profile page looks good" | Subjective - what is "good"? |
| "Form is user-friendly" | Vague - can't be tested |
| "Performance is acceptable" | No measurable threshold |
| "Errors are handled" | Doesn't specify which errors or how |
| "The code is clean" | Opinion-based, not testable |

### Good Criteria (Use These)

| Good | Why It Works |
|------|--------------|
| "Profile page displays user name, email, and avatar" | Specific elements to verify |
| "Form shows inline errors for invalid inputs" | Observable behavior |
| "Page loads in under 3 seconds" | Measurable threshold |
| "Invalid email shows 'Please enter a valid email'" | Exact error message |
| "Build passes with no TypeScript errors" | Binary pass/fail |

## Criterion Structure

Write each criterion as a checkbox item that can be marked complete:

```markdown
## Done Criteria
- [ ] [Action/State] [Observable result]
```

Examples:
```markdown
- [ ] Login form accepts valid email and password
- [ ] Successful login redirects to dashboard page
- [ ] Invalid credentials show "Invalid email or password" error
- [ ] Empty email field shows "Email is required" error
- [ ] Loading spinner visible during login request
- [ ] Build completes without errors
```

## Recommended Criteria Count

- **Simple features**: 3-5 criteria
- **Medium features**: 5-8 criteria
- **Complex features**: 8-12 criteria

If you have more than 12 criteria, consider breaking the feature into smaller tasks.

## Criteria Checklist

Before finalizing, verify each criterion passes this checklist:

1. **Binary result**: Can it be marked pass or fail? (no partial credit)
2. **No implementation details**: Does it describe WHAT, not HOW?
3. **Testable in browser**: Can the verification agent check this?
4. **Independent**: Can it be verified without checking other criteria first?
5. **Complete sentence**: Does it clearly describe the expected behavior?

## Hybrid Testing Approach

The verification agent uses two approaches:

### Interactive Testing (for functional criteria)
- Click buttons, fill forms, navigate pages
- Verify behavior through element state changes
- Check network requests and responses

### Screenshot Testing (for visual criteria)
- Capture page state at key moments
- Verify presence of visual elements
- Check layout and positioning

When writing criteria, consider which approach will verify it:
- "Button changes to 'Saving...' when clicked" → Interactive
- "Page header shows user avatar in top right" → Screenshot
- "Form validation errors appear below inputs" → Both

## Common Criteria Templates

### Form Feature
```markdown
- [ ] Form displays all required input fields
- [ ] Submit button is disabled until form is valid
- [ ] Valid submission shows success message
- [ ] Invalid [field] shows "[error message]" error
- [ ] Build completes without errors
```

### List/Display Feature
```markdown
- [ ] Page displays list of [items]
- [ ] Each item shows [required fields]
- [ ] Empty state shows "[message]" when no items
- [ ] Loading spinner visible while fetching
- [ ] Build completes without errors
```

### CRUD Feature
```markdown
- [ ] User can create new [item]
- [ ] User can view [item] details
- [ ] User can edit existing [item]
- [ ] User can delete [item]
- [ ] Confirmation dialog appears before delete
- [ ] Build completes without errors
```
