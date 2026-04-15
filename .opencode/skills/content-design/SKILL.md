---
name: content-design
description: Design UX writing, microcopy, tone of voice, informational hierarchy, and content strategy so every word in the UI is intentional, clear, and helpful
---

## What I do

I design the content layer of UI — the words, structure, and information architecture that make interfaces understandable:

- **UX writing** — Button labels, headings, descriptions, form labels, error messages
- **Microcopy** — Tiny text that guides behavior (tooltips, placeholders, helper text, empty states)
- **Tone of voice** — Consistent personality across all UI text
- **Informational hierarchy** — What to say, what to emphasize, what to omit
- **Content patterns** — Error messages, success messages, confirmations, onboarding copy
- **Naming** — Feature names, navigation labels, settings terminology

## When to use me

Use this skill when:
- Writing or reviewing any UI text (buttons, labels, headings, descriptions)
- Defining the tone of voice for a product
- Crafting error messages that help rather than frustrate
- Designing onboarding flows, empty states, or confirmation dialogs
- Naming features, navigation items, or settings
- Reviewing copy for clarity, consistency, and tone

## How I work

1. **Define the content principles** — What does this product sound like? How does it treat the user?
2. **Establish tone of voice** — Choose a personality and write within it consistently.
3. **Map the content hierarchy** — Every screen has a primary message. Define it before writing anything.
4. **Write concisely** — Cut ruthlessly. If a word doesn't add meaning, remove it.
5. **Test clarity** — Can a first-time user understand this in 3 seconds? If not, rewrite.
6. **Maintain a glossary** — Track every term used in the UI for consistency.

## Content principles

Every piece of UI text should satisfy these four principles:

### 1. Clear
The user understands it on first read. No ambiguity.

| Unclear | Clear |
|---------|-------|
| "Submit your information" | "Save profile" |
| "An error occurred" | "Password must be at least 8 characters" |
| "Configure your preferences" | "Settings" |
| "Proceed with the operation" | "Continue" |

Rules:
- Use the user's vocabulary, not the system's
- Prefer short words over long ones (use → utilize, start → initiate, end → terminate)
- One idea per sentence
- Active voice: "You can add team members" not "Team members can be added"

### 2. Concise
Every word earns its place. Respect the user's time and screen space.

| Verbose | Concise |
|---------|---------|
| "Click the button below to create a new project" | "Create project" |
| "Are you sure you want to delete this item? This action cannot be undone." | "Delete this item? This can't be undone." |
| "In order to continue, you must first" | "To continue," |
| "You have successfully created your account!" | "Account created!" |

Rules:
- Maximum 3 words for button labels
- Maximum 8 words for headings
- Maximum 40 words for descriptions
- Maximum 15 words for error messages
- If you can cut a word without losing meaning, cut it
- "In order to" → "To", "the fact that" → eliminate, "at this point in time" → "now"

### 3. Consistent
Same thing = same word. Every time.

| Inconsistent | Consistent |
|--------------|------------|
| "Delete / Remove / Erase" | Pick one, use always (usually "Delete") |
| "Settings / Preferences / Options" | Pick one (usually "Settings") |
| "Sign in / Log in / Login" | Pick one, use always |
| "Team / Group / Members" | Pick one, define it, use always |

Rules:
- Create a glossary of product terms
- Never use two words for the same concept
- Tense consistency: use present tense for descriptions, imperative for actions
- Capitalization consistency: sentence case for all UI text (Title Case only for proper nouns)

### 4. Helpful
The text helps the user succeed. It tells them what to do, not just what happened.

| Unhelpful | Helpful |
|-----------|---------|
| "Invalid input" | "Enter a valid email address" |
| "Error 5003" | "Couldn't save changes. Try again." |
| "No results" | "No matching projects. Try a different search or create one." |
| "Operation successful" | "Project saved. View it in your dashboard." |

Rules:
- Always suggest a next action
- Error messages: say what happened, why, and what to do about it
- Success messages: confirm the outcome and suggest what's next
- Never blame the user ("You entered an invalid email" → "Enter a valid email address")

## Tone of voice

Define the product's personality with 3-4 attributes. Examples:

### Professional / Trustworthy (B2B, finance, healthcare)
- Attributes: Confident, Clear, Respectful, Measured
- Writing: Direct, factual, no exclamation marks, no slang
- Example: "Your report is ready. Download it to review the full analysis."

### Friendly / Approachable (Consumer, social, lifestyle)
- Attributes: Warm, Helpful, Casual, Encouraging
- Writing: Contractions ("you'll"), occasional exclamation, conversational
- Example: "Your report's ready! Grab it and check out the highlights."

### Technical / Precise (Developer tools, infrastructure)
- Attributes: Exact, Efficient, Neutral, Unambiguous
- Writing: Technical terms, no adjectives, terse phrasing, code-style
- Example: "Report generated. `report_2026_04.csv` available for download."

### Playful / Energetic (Games, creative tools, kids)
- Attributes: Fun, Enthusiastic, Witty, Dynamic
- Writing: Emojis occasionally, wordplay, exclamation marks, humor
- Example: "Boom — your report is served! 🎉 Time to dig in."

**Tone scales by context**: Even within a product, tone varies:
| Context | Tone shift |
|---------|-----------|
| Onboarding | Warmer, more encouraging, more words |
| Error messages | Softer, more helpful, less casual |
| Destructive actions | Serious, no humor, clear consequences |
| Empty states | Encouraging, forward-looking |
| Success confirmations | Brief, positive, suggest next step |
| Loading states | Reassuring, show progress |
| Settings/technical | Precise, neutral, less casual |

## Content patterns

### Button labels

```
Pattern: [Verb] + [Noun] (imperative mood)

✓ Create project    ✗ New project
✓ Save changes      ✗ Apply
✓ Delete file       ✗ Remove
✓ Send message      ✗ Submit
✓ Import data       ✗ Upload
✓ Export CSV        ✗ Download data as CSV
✓ Generate report   ✗ Run
✓ Cancel            ✗ Abort
```

Rules:
- Always start with a verb (except "Cancel", "OK", "Back", "Next")
- 1-3 words maximum
- Primary action: specific verb ("Save", "Publish", "Send")
- Destructive action: name the thing being destroyed ("Delete account", not "Confirm")
- Cancel never needs a noun — it's always just "Cancel"
- Never use "Submit" — it's vague. Say what happens: "Save", "Send", "Publish"

### Headings and page titles

```
Pattern: [Noun phrase] or [Noun + prepositional phrase]

✓ Project settings     ✗ Configure your project
✓ Team members         ✗ Managing team members
✓ Billing history       ✗ Your billing history overview
✓ API keys             ✗ API key management
```

Rules:
- Noun phrases, not sentences or verbs
- No articles (the, a, an) unless necessary for clarity
- 2-4 words
- Sentence case, not Title Case (except proper nouns)
- Match the navigation label exactly

### Form labels and helper text

```
Label: Email address
Helper: We'll only use this for account recovery.
Placeholder: name@example.com

Label: Password
Helper: At least 8 characters with one number.
Placeholder: (no placeholder — use helper text instead)

Label: Project name
Helper: This will be visible to your team.
Placeholder: My awesome project
```

Rules:
- Labels are 1-3 words, noun phrases
- Helper text adds context the label can't: "why" not "what"
- Placeholders show format/examples, not instructions
- Never put critical instructions in placeholders — they disappear on input
- Group label + input + helper as a unit
- Required fields: mark with *, explain at top: "* Required"
- Optional fields: mark as "(optional)" in the label, not required fields as "(required)"

### Error messages

The three-part error message structure:

```
1. What happened: "Password is too short"
2. Why it matters: "Use at least 8 characters for security"
3. What to do: "Add 3 more characters"
```

In practice, combine when possible:

| Context | Error message |
|---------|--------------|
| Required field | "Enter an email address" |
| Format validation | "Enter a valid email address" |
| Length validation | "Password must be at least 8 characters" |
| Uniqueness constraint | "This email is already registered. Sign in or use a different email." |
| Server error | "Couldn't save changes. Try again." |
| Network error | "No internet connection. Check your network and try again." |
| Permission error | "You don't have permission to edit this. Contact your admin." |
| Rate limit | "Too many attempts. Try again in 5 minutes." |
| Destructive confirmation | "Delete ‘Project Name'? This can't be undone." |

Rules:
- Never say "Error" or show error codes to users
- Never blame the user — reframe as instructions, not accusations
- Be specific about what's wrong and how to fix it
- If you can't suggest a fix, provide a way to get help (support link)
- Show errors below the field, aligned left
- Clear the error when the user starts typing
- Re-validate on blur, not on every keystroke

### Success messages

```
Pattern: [Confirmation] + [Next action]

✓ "Project created. Add your first task."
✓ "Changes saved."
✓ "Email sent to 3 team members."
✓ "Account updated. Your changes are live."
```

Rules:
- Brief: 2-6 words
- Confirms what happened, not just "Success!"
- Suggests a logical next step when relevant
- Use toast for reversible actions (save, update, send)
- Use a full page or section for irreversible milestones (account created, payment processed)
- Never use "Successfully" — it's redundant. "Saved" not "Successfully saved"

### Confirmation dialogs

```
┌──────────────────────────────────────────┐
│ Delete project?                          │  ← Title: action as a question
│                                          │
│ This will permanently delete "Project    │  ← Body: consequence + scope
│ Name" and all 47 tasks. This can't be    │
│ undone.                                  │
│                                          │
│  [Cancel]              [Delete project]   │  ← Buttons: neutral + destructive
└──────────────────────────────────────────┘
```

Rules:
- Title: the action as a question ("Delete project?", "Leave without saving?")
- Body: what will happen, scope of the action, irreversibility
- Confirm button: labels the destructive action ("Delete", not "Confirm")
- Cancel button: always available, never colored
- Never use "Are you sure?" — it's redundant and condescending

### Empty states

```
┌──────────────────────────────┐
│                               │
│     [Illustration]            │
│                               │
│   No projects yet             │  ← Heading: state name
│                               │
│   Create your first project   │  ← Description: what to do
│   to start tracking tasks.    │
│                               │
│   [Create project]            │  ← CTA: primary action
│   Import from CSV             │  ← Secondary: alternative
│                               │
└──────────────────────────────┘
```

Rules:
- Heading: 2-4 words stating the state ("No projects yet", "No results")
- Description: 1-2 sentences, encouraging, explains what to do
- CTA: The most common action to fill the empty state
- Secondary link: alternative action (import, browse, search)
- Never show a blank page with only text — the illustration adds warmth
- Never blame the user ("You haven't created anything") — reframe positively ("Create your first project")

### Loading states

| Duration | Pattern | Copy |
|----------|---------|------|
| < 1s | Inline spinner (no text) | — |
| 1-3s | Spinner + text | "Saving..." / "Loading..." |
| 3-10s | Progress bar + text | "Generating report..." / "Uploading 3 of 8 files..." |
| > 10s | Skeleton screen | No copy needed — the skeleton IS the message |
| Unknown | Skeleton screen | Same as above |

Rules:
- Use present participle: "Saving...", "Loading...", "Generating..."
- Be specific when possible: "Uploading report.csv..." not "Processing..."
- Never promise a time estimate you can't guarantee
- If a process takes >30s, add a cancellation option
- Skeleton screens use no text — they show shape, not words

### Onboarding copy

```
┌──────────────────────────────┐
│        [Illustration]         │
│                               │
│   Step 1 of 3                 │  ← Progress indicator
│                               │
│   Organize your work          │  ← Heading: benefit (not feature)
│                               │
│   Use projects to group       │  ← Description: how to achieve the benefit
│   related tasks and track      │
│   progress as a team.         │
│                               │
│   [Next]                      │  ← Action
│   Skip for now                │  ← Escape hatch
└──────────────────────────────┘
```

Rules:
- Headings describe benefits, not features ("Organize your work" not "Projects")
- 1-2 sentences per step
- Maximum 4 steps, 3 is ideal
- Always provide "Skip" — not everyone needs onboarding
- Show progress clearly (step dots, progress bar)
- End with a clear success state and first action

## Navigation and naming

### Navigation labels

| Good | Why | Bad | Why |
|------|-----|-----|-----|
| Projects | Noun, clear | My Projects | "My" is redundant in your own nav |
| Settings | Standard | Config | Jargon |
| Team | Short | Team Members | Too long for nav |
| Billing | Specific | Account & Billing | Split them or use the primary |
| API | Standard | Developer API | "Developer" is redundant |

Rules:
- 1-2 words per nav item (3 maximum)
- Nouns, not verbs (Projects, not Manage Projects)
- Use established conventions (Settings, not Preferences or Options)
- Same label in navigation, page heading, and breadcrumbs
- Avoid brand-specific names for generic features

### Feature naming

Rules:
- Use the user's mental model, not the system architecture
- "Projects" not "Workspaces" (unless workspace is the established metaphor)
- "Tasks" not "Action Items" or "Todos" (unless brand-specific)
- "Members" not "Users" (humanizing, avoids admin-speak)
- "Templates" not "Presets" or "Blueprints"
- Check competitors: if everyone calls it X, call it X (learnability > originality)

## Content style guide

### Capitalization
- **Sentence case** for all UI text: "Save changes", "Team members", "Create project"
- **Title case** only for: proper nouns ("Google Drive"), brand names ("GitHub")
- **All caps** only for: badges ("NEW", "PRO"), not for emphasis
- **No capitalization** for: feature names used generically ("api", not "API" — unless it's an acronym)

### Punctuation
- **No periods** at the end of: headings, labels, buttons, toasts, list items
- **Periods** at the end of: body paragraphs, descriptions longer than one sentence
- **No exclamation marks** except in: playful brands, onboarding celebrations
- **Oxford comma**: yes ("Create, edit, and delete projects")
- **Ellipsis**: for in-progress actions ("Saving..."), not for trailing off

### Numbers
- Spell out: one through nine ("Add up to 5 files")
- Use numerals: 10+ ("Add up to 12 files")
- Always use numerals: ages, dates, measurements, percentages ("3 days ago", "5%", "2 GB")
- Use tabular figures for all numbers in data columns and dashboards

### Pronouns
- Avoid "you" and "your" when possible: "Save changes" not "Save your changes"
- Use "you" when clarifying agency: "You can add up to 5 files"
- Never use "we" for system actions: "Your settings were saved" not "We saved your settings"
- Use "this" to reference: "Delete this project" not "Delete the project"

## Glossary creation

Create a term glossary for every product:

```markdown
| Term | Definition | Used in | Do not use |
|------|-----------|---------|------------|
| Project | A collection of tasks organized around a goal | Nav, headings, empty states | Workspace, Board |
| Task | An actionable item within a project | Lists, forms, filters | Action item, Todo, Ticket |
| Member | A person on the team | Settings, billing, nav | User, Account |
| Owner | The person who created and manages a project | Project header, permissions | Admin, Creator |
| Archive | Hide without deleting | Project menu, filters | Delete, Remove, Hide |
```

Rules:
- Every concept has exactly one term
- Record the term, its user-facing definition, where it appears, and what NOT to call it
- Review the glossary when adding new features
- If two terms mean the same thing, choose one and eliminate the other

## Quality checklist

- [ ] Every button uses a verb + noun label (1-3 words)
- [ ] Every error message explains what happened and how to fix it
- [ ] Every empty state has a heading, description, and CTA
- [ ] Tone of voice is defined and followed across all UI text
- [ ] No two terms refer to the same concept (glossary exists)
- [ ] No placeholders contain critical instructions
- [ ] Headlines are noun phrases, buttons are imperative verbs
- [ ] Sentence case throughout (no Title Case)
- [ ] No exclamation marks unless the brand tone supports them
- [ ] Confirmation dialogs show consequences and have Cancel
- [ ] Success messages confirm what happened and suggest next action
- [ ] Loading messages are specific ("Generating report..." not "Loading...")
- [ ] Destructive actions are named explicitly ("Delete project" not "Confirm")
- [ ] Text is 30-50% shorter than initial draft (cut ruthlessly)
- [ ] Numbers use consistent formatting (tabular figures, compact notation)

## Anti-patterns I avoid

- "Click here" or "Learn more" as link text — the link should describe its target
- "Are you sure?" as a confirmation — condescending; state the consequence instead
- Humor in error messages — the user is already frustrated
- Jargon and technical terms in consumer UI — use simple language
- "No data" or "Nothing to show" as empty states — provide guidance
- Long paragraphs in UI — break into bullet points or shorter sentences
- Vague labels ("Submit", "Apply", "OK") — use specific verbs
- All-caps for emphasis — use weight or color instead
- Inconsistent verb tense — use imperative for actions, present for states
- "Or" choices in buttons — design so one path is clearly primary
- Marketing copy in the product UI — marketing and product copy are different disciplines