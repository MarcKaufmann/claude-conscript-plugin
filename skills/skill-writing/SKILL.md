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

### 2. Assess and match quality style
Read the other skills in the plugin before writing. If unsure what
style or quality to follow, ask or do a standalone analysis of the
different styles present. Not all existing skills are worth emulating
— identify the best-written ones and match those.

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
If the skill's audience includes non-experts, define plugin-specific
terms on first use. Domain experts who write skills often omit
definitions that newcomers need.

### 6. Verify claims about frameworks
Before writing "the framework can't do X," check the source code.
Wrong capability claims in skills lead to wrong code and misdirected
development effort.

