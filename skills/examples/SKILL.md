---
name: examples
description: |
  Reference for finding and understanding example conscript studies.
  Reads example locations from the project's study-config.md. Use when
  writing a new study or looking for patterns to follow.
user-invocable: false
---

Guide for finding and using example studies as references.

## Locating Examples

### Built-in congame examples

The congame repository contains small, focused examples in
`conscript/examples/`. Look for `study-config.md` in the current
project to find the congame path. If not found, ask the user.

Key files and what they demonstrate:

| File | Pattern |
|------|---------|
| `kitchen-sink.rkt` | Full study flow: info pages, forms, consent branching, nested `defstudy` |
| `form.rkt` | Minimal form: text + textarea inputs, submit handler |
| `tutorial.rkt` (in `congame-doc/conscript-examples/`) | Simple survey: form → variable interpolation → thank you |
| `with-require.rkt` | Using `#lang conscript/with-require` for external modules |

### Project-specific examples

Check `study-config.md` in the current project for paths to other
repositories with example studies. Common locations:

- The current project itself (look for `.rkt` files in `studies/` or
  `experiments/`)
- Other research project repos the user has listed

### Pattern index

When looking for a specific pattern, search examples in this order:

| Pattern needed | Where to look first |
|----------------|---------------------|
| Basic form with inputs | congame `form.rkt` |
| Info page with button | congame `kitchen-sink.rkt` |
| Consent with branching | congame `kitchen-sink.rkt` |
| Nested/child study | congame `kitchen-sink.rkt` |
| Looping over rounds | look for `add1 rounds` in project examples |
| Treatment assignment (balanced) | look for `shuffle` or `assigning-treatments` in project examples |
| Conditional branching on treatment | look for `if competition?` in project examples |
| Matchmaking / pairing | look for `make-matchmaker` in project examples |
| Wait pages with refresh | look for `refresh-every` in project examples |
| Slider inputs | look for `make-sliders` or `input-range` in project examples |
| Multiple checkboxes | look for `make-multiple-checkboxes` in project examples |
| Timers | look for `@timer` in project examples |
| Currency formatting | look for `~pound` or `~$` in project examples |
| Bot testing | look for `make-bot-model` in project examples |
| Autofill metadata | look for `make-autofill-meta` in project examples |

### How to use examples

1. Read `study-config.md` to get repo paths
2. Identify the pattern needed
3. Use the pattern index above to find the right file
4. Read the relevant section of the file (not necessarily the whole thing)
5. Adapt the pattern to the current study — don't copy verbatim, match
   the project's conventions from `study-config.md`
