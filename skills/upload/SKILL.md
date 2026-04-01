---
name: upload
description: |
  Upload a study to a congame server. Use when the user wants to deploy
  or upload a study file to local, staging, or production.
argument-hint: "<study-id> <path> [server]"
allowed-tools: Bash(raco congame *)
---

Upload a study to a congame server using `raco congame upload`.

## Usage

```
raco congame upload <study-id> <path>
```

- `<study-id>` — the study identifier (e.g. `pcs27`, `teq73`)
- `<path>` — path to the `.rkt` file (e.g. `experiments/PCS27.rkt`)

## Resolving arguments

When arguments are not fully provided, infer them from context:

- If only a study ID is given (e.g. `/upload PCS27`), find the matching
  `.rkt` file in `experiments/` (case-insensitive match).
- If only a file path is given (e.g. `/upload experiments/PCS27.rkt`),
  read the file and look at the `(provide ...)` form to find the study
  ID(s) it exports. Prefer the `-with-admin` variant if one is
  exported (see below).
- If no arguments are given, look at what the user was just working on
  — the most recently discussed or edited study file — and confirm
  before uploading.
- If the argument is ambiguous or no match is found, ask the user to
  clarify.

## Prefer `-with-admin` variants

When resolving the study ID, always read the `(provide ...)` form in the
file. If a `-with-admin` variant is exported (e.g. `PCS27-with-admin`),
upload that instead of the base study. The `-with-admin` variant
includes bot models for testing and is a strict superset of the base
study.

Only upload the base study if no `-with-admin` variant exists.

## Steps

1. Ensure the user is logged in. If not, run `raco congame login`
   which prompts for a server URL (default `http://127.0.0.1:5100`)
   and opens a browser for authentication.
2. Read the file's `(provide ...)` form and pick the `-with-admin`
   variant if available.
3. Run `raco congame upload <study-id> <path>`.

## Servers

See the `docs` skill.

If the user specifies a server (third argument or by name like "staging"
or "production"), they may need to `raco congame login` to that server
first.

## Testing after upload

After uploading, the study can be tested at:
`<server>/_anon-login/<study-id>`
