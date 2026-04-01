---
name: racket-coding
description: |
  Racket language reference for at-expression syntax, data types,
  control flow, pattern matching, and idioms. Use when writing or reading
  Racket code in this project.
user-invocable: false
---

Racket patterns and idioms as used in this codebase. All study code
uses `#lang conscript`, which is a restricted subset of Racket with
at-expression syntax enabled.

## Imports

- Always sort imports alphabetically.

## At-Expression Syntax

Documentation: https://docs.racket-lang.org/scribble/reader.html#(part._reader)

The `@` reader is the primary syntax for mixing code and content.
Understanding it is essential.

```racket
@function-name[arg1 arg2]{text body}
;; equivalent to: (function-name arg1 arg2 "text body")

@function-name{text body}
;; equivalent to: (function-name "text body")

@function-name[arg1 arg2]
;; equivalent to: (function-name arg1 arg2)

@|variable-name|        ;; splice variable value into text
@(expression)           ;; splice expression result into text
@;comment               ;; at-expression line comment
```

### Nesting

```racket
@md{# Title
    This is @strong{bold} and @(~a (+ 1 2)) is three.
    @button{Continue}}
```

At-expressions can nest arbitrarily. Indentation in `{...}` is
significant — the common leading whitespace is stripped.

### `@md*{}` vs `@md{}`

- `@md{...}` — renders markdown and wraps in a container div (full page)
- `@md*{...}` — renders markdown as a fragment (for embedding, not standalone)
