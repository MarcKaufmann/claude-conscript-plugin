---
name: create-study
description: |
  Create a new conscript study. On first use in a project, runs
  initialization to learn the project's study patterns and example
  locations. On subsequent uses, generates a study .rkt file matching the
  project's conventions.
argument-hint: "[study-description]"
---

Create a new conscript study for the current project.

Before starting, load the `coding`, `racket-coding`, and
`conscript-coding` skills for reference.

## Phase A: Project Initialization

Check if a `study-config.md` file exists in the current working
directory (or in `.claude/`). If it does, skip to Phase B.

If no config exists, run initialization:

### Step 1: Check for existing config

Ask the user:

> Do you already have a .md file describing the type of studies you
> build in this project? If so, provide the path.

If yes, read the file and use it as the project config. Copy or
symlink it to `study-config.md` in the project root.

### Step 2: Ask about study patterns

If no existing config, ask these questions (adapt based on answers):

1. **Domain:** What research area are these studies for? (e.g., belief
   updating, social preferences, competition and morality)
2. **Common structure:** What does a typical study look like?
   - Number of rounds/repetitions?
   - Treatment arms (and how assigned)?
   - Typical measurements (scales, free text, choices)?
3. **Constraints:** Any standard requirements?
   - Consent text or IRB requirements?
   - Payment structure (flat fee, performance-based, lottery)?
   - Required demographic questions?
   - CSS/branding requirements?
4. **Naming:** Any naming conventions for study IDs?
   (e.g., 5-char alphanumeric like PCS27, or descriptive like
   urn-beliefs)

### Step 3: Ask about example locations

Ask:

> Where is your congame repository cloned? (e.g., ~/projects/congame)
> This gives me access to framework examples.
>
> Are there other repositories with example studies I should learn from?
> (e.g., ~/projects/many-designs, ~/projects/long-term-beliefs)

### Step 4: Generate config

Write `study-config.md` in the project root with:

```markdown
# Study Configuration

## Domain
[from answers]

## Typical Study Structure
[from answers]

## Constraints
[from answers]

## Naming Conventions
[from answers]

## Example Repositories
- congame: [path]
- [other repos listed by user]
```

Confirm with the user before saving.

## Phase B: Create Study

### Step 1: Read config

Read `study-config.md` from the project root. Also load the `examples`
skill to know where to find reference code.

### Step 2: Understand the study

If `$ARGUMENTS` contains a description, use it. Otherwise ask:

1. What is the study about? What is the research question?
2. How does it differ from the typical structure in the config?
   (e.g., different number of rounds, new treatment arms, different
   measurements)
3. Any new elements not covered by the config? (e.g., matchmaking,
   timers, special widgets)

### Step 3: Find relevant examples

Based on the study design, identify the most relevant existing studies
from the example repositories listed in `study-config.md`. Read them
for reference patterns. Prioritize:

- congame/conscript/examples/ for basic patterns
- Project-specific examples for domain patterns
- Other repos for advanced patterns (matchmaking, complex treatments)

### Step 4: Generate the study

Write a `.rkt` file following:

1. `#lang conscript` header
2. Imports (only whitelisted modules, sorted alphabetically)
3. `(provide study-name study-name-with-admin)`
4. CSS (`common-styles` — copy from config or existing study)
5. Variables (`defvar` for participant, `defvar/instance` for shared)
6. Steps (following patterns from conscript-coding skill)
7. Study flow (`defstudy` with connected graph)
8. Bot models and `-with-admin` variant

Apply all conventions from the project config and the `coding` skill
checklist.

### Step 5: Review

After generating, run through the review checklist from `study-review`
internally. Fix any issues before presenting to the user.

### Step 6: Present and iterate

Show the generated study to the user. Ask if they want changes.
Iterate until satisfied.

Offer to upload via the `upload` skill when ready.
