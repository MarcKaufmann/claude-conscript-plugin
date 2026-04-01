---
name: docs
description: |
  Project documentation for the Many Designs replication study. Covers
  repository layout, resource links, study progress tracking, and known
  issues. Use when navigating the repo, looking up study status, or
  understanding project structure.
user-invocable: false
---

## Project Overview

Behavioral economics research project ("Many Designs Replication")
implementing ~40 experimental study designs in Racket using the
**congame** framework (`conscript`). Each study replicates a published
experiment, typically comparing a competition treatment against a
control condition.

- **Framework docs:** https://joeldueck.com/what-about/congame
- **Framework source:** https://github.com/MarcKaufmann/congame
- **Production:** https://totalinsightmanagement.com
- **Staging:** https://staging.totalinsightmanagement.com
- **Local testing:** `http://127.0.0.1:5100/_anon-login/<study-id>` (e.g. `ach91`)
