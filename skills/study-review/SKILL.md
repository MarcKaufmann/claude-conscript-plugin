---
name: study-review
description: |
  Review a study for correctness and coding patterns, then upload and
  test it. Use when the user wants to review, verify, or QA a study
  before or after deployment.
argument-hint: "[study-file]"
allowed-tools: Bash(raco congame *)
---

Review a study file for correctness, upload it, and run a test session.

Before starting, load the `coding`, `racket-coding`, and
`conscript-coding` skills for reference.

## 1. Identify the study

- If a file path is provided as `$ARGUMENTS`, use that.
- Otherwise, check conversation context for a recently discussed or
  edited study.
- If still unclear, ask the user which study to review.

Read the study file fully before proceeding.

## 2. Understand the study design

Check whether the conversation already contains context about the
study's design. If not, ask the user:

- What is the experimental task? (e.g. matrix task, die roll, public
  goods game)
- What are the treatments? (e.g. competition vs. control)
- What is the outcome measure? (e.g. claimed score minus true score)
- What is the payment structure?

Use their answers to verify that the code matches the intended design.

## 3. Code review

Review the study code against the following checklist. Report each item
as passing or failing, with specifics for any failures.

### Structure
- [ ] File starts with `#lang conscript`
- [ ] `(provide ...)` exports the study and its `-with-admin` variant
- [ ] `(require ...)` only uses whitelisted modules (see
  `conscript-coding` skill)
- [ ] `common-styles` CSS is defined and included in every `@md{...}`
  step

### Variables
- [ ] All participant state uses `defvar`
- [ ] All shared state uses `defvar/instance`
- [ ] Every mutation of a `defvar/instance` variable is inside
  `with-study-transaction`

### Treatment assignment
- [ ] Uses balanced shuffle pattern (groups of 2 or 4) or
  `assigning-treatments`
- [ ] Treatment assignment step calls `(skip)` at the end
- [ ] `competition?` or equivalent flag is a `defvar`, not
  `defvar/instance`

### Study flow
- [ ] `defstudy` forms a connected graph — no orphaned steps
- [ ] Study terminates with `,(lambda () done)`
- [ ] Branching on treatment uses `,(lambda () (if competition? ...))`
- [ ] Looping patterns reset the counter when exiting the loop
- [ ] Computation-only steps (init, assign) call `(skip)`

### Forms
- [ ] All required fields use `(ensure binding/... (required))`
- [ ] Form render functions use `@md*{...}` (fragment), not `@md{...}`
- [ ] `@|submit-button|` is present in every form render function

### Matchmaking (if applicable)
- [ ] `make-matchmaker` is called once at module level, not inside a
  step
- [ ] Wait page uses `@refresh-every[...]`
- [ ] Opponent data retrieval checks for `#f` before displaying results
- [ ] Wait page includes `@meta[#:name "wait"]` for bot detection

### Bot testing
- [ ] `make-admin-study` wraps the main study with `#:models`
- [ ] Bot model has a clause for every step in the study
- [ ] Wait/refresh steps use `(void)` in the bot model
- [ ] Final step uses `(bot:completer)`
- [ ] Forms have `make-autofill-meta` with presets matching bot model
  kinds

### Content
- [ ] Instructions match the intended experimental design (verify
  against user's answers from step 2)
- [ ] Payment amounts and descriptions are consistent throughout
- [ ] Attention check text and expected answer are correct

## 4. Ensure `-with-admin` variant exists

If the study is missing a `-with-admin` variant with bot models, add
one before reporting findings. This is required infrastructure, not
optional — do not ask the user for permission, just add it.

### What to add

1. **Requires:** ensure `conscript/admin` and `racket/match` are in the
   `(require ...)` form.

2. **Provide:** add the `-with-admin` identifier to the `(provide ...)`
   form (e.g. `NJJ10-with-admin`).

3. **`@meta[#:name "wait"]`:** add to every wait/refresh page (both
   matchmaking wait pages and opponent-polling pages). Bots use this
   marker to detect whether they are still waiting.

4. **`make-autofill-meta`:** add to every form render function. Define
   presets for two bot kinds that exercise different code paths (e.g.
   `'discloser` / `'non-discloser`, `'liar` / `'honest`). Values
   should be valid strings matching the form's expected input (radio
   option values, number strings for number inputs, etc.).

5. **Bot model:** define a `(make-bot-model kind)` function that
   dispatches on step path using `match`. It must handle every step:
   - Button/content pages → `(bot:continuer)`
   - Form pages → `(bot:autofill kind)`
   - Wait/refresh pages → check `(bot:find "meta[name=wait]")`: if
     found `(void)`, otherwise `(bot:continuer)`
   - Computation-only steps (skip) → `(bot)` (default case handles
     these)
   - Final step → `(bot:completer)`

6. **`make-admin-study`:** register the admin study with both bot
   kinds:
   ```racket
   (define STUDY-with-admin
     (make-admin-study
      #:models
      `((kind-a . ,(make-bot-model 'kind-a))
        (kind-b . ,(make-bot-model 'kind-b)))
      STUDY))
   ```

### Reference

See the `coding` and `conscript-coding` skills for the exact patterns
and bot action reference.

## 5. Report findings

Present the review as a checklist with pass/fail status. For any
failures, explain the issue and suggest a fix. Ask the user if they want
you to apply the fixes before proceeding to upload.

## 6. Upload

After the review passes (or fixes are applied), upload the study using
the `upload` skill:

```
raco congame upload <study-id> <path>
```

Read the `(provide ...)` form in the file to determine the study ID.

## 7. Test

**Important:** Before running simulations, remind the user that if this
is the first time the study has been uploaded, they need to create a
study instance on the server first (via the admin UI at
`<server>/admin`). The simulation will fail without an active instance.
Wait for the user to confirm before proceeding.

After the instance is ready, ask the user to manually test the study.

**Note:** `raco congame simulate -n <N> <study-id>` opens browser
windows but still requires manual interaction — it does NOT run bots
automatically. Do not use it for unattended testing.

Provide the user with the test URL:
`<server>/_anon-login/<study-id>` (default server:
`http://127.0.0.1:5100`)

For studies with matchmaking, remind the user to open multiple
sessions (one per group size) to test the pairing flow.
