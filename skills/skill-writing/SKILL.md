---
name: skill-writing
description: |
  Lessons for writing Claude Code skills, learned from adversarial review
  of the create-study skill. Currently tailored to congame/conscript
  plugin skills — may generalize to other plugins but not verified.
user-invocable: false
---

**Scope caveat:** These lessons come from writing skills for the
congame/conscript plugin. They likely generalize to other Claude Code
plugins but have not been tested outside this context.

## Lessons

### 1. Don't restate loaded skills
If your skill says "load skill X for reference," don't repeat X's
content. The model has it. Restating creates duplication that drifts
when the original is updated.

### 2. Match the existing style
Read the other skills in the plugin before writing. Match their
density, structure, and tone. If existing skills use flat numbered
sections, don't introduce "Phase A / Phase B" labels. Caveat: if
existing skills are poorly written, matching them propagates defects.

### 3. Don't over-script interviews
If the skill needs to ask the user questions, state the intent ("learn
about their project's structure and constraints") not a rigid script
with numbered sub-questions. The model asks better questions when given
intent than when given a script.

### 4. Don't force two invocations
If a skill has a setup step (e.g., writing a config file), continue
into the main task after setup. Don't stop and tell the user to
re-invoke. Caveat: if the setup requires user input that blocks
further progress (e.g., a credential), pausing is legitimate.

### 5. Define jargon for newcomers
If the skill's audience includes non-experts, define domain terms
(e.g., "conscript," "bot model," "-with-admin variant"). Existing
skills written by domain experts often omit these.

### 6. Separate reference from report
Reference docs (.md) for tools to read should be dense and structured.
Reports (.qmd) for humans to read should have prose and context. Don't
write one document trying to serve both.

### 7. Verify claims about frameworks
Before writing "the framework can't do X," check the source code.
The initial bot testing analysis incorrectly claimed bots can't read
pages — they use Marionette/Firefox with full DOM access. Wrong claims
in skills lead to wrong code.
