# Bot Testing vs Playwright Testing: Detailed Analysis (Corrected)

## Correction Note

The initial version incorrectly stated bots run server-side without browser access. Congame bots use Marionette to drive headless Firefox. This document is fully revised.

## How Congame Bots Actually Work

Bots are defined as functions dispatching on step path, but they execute inside a **real headless Firefox browser** via the Marionette protocol.

### Architecture (from `congame-core/components/bot.rkt`)

```racket
(define/contract (run-bot b
                          #:study-url url
                          #:username username
                          #:password password
                          #:delay [delay 0]
                          #:browser [browser #f]
                          #:headless? [headless? #t]
                          #:port [port #f])
```

- Default: headless Firefox (`#:headless? #t`)
- Uses `call-with-marionette/browser/page!` to create a browser session
- Bot navigates to the study URL, logs in, and walks through steps
- Each step: bot model function receives the step path and a `bot` continuation
- Bot actions operate on real DOM elements

### Bot Actions (from `congame-core/components/bot.rkt`)

| Action | Implementation | DOM access? |
|---|---|---|
| `(bot:continuer)` | `(element-click! (find-widget))` — clicks the `.next-button` element | Yes |
| `(bot:completer)` | `(element-click! (find "a.button.complete-button"))` | Yes |
| `(bot:autofill kind)` | Reads `make-autofill-meta` JSON from page, fills form fields | Yes |
| `(bot:click id)` | `(element-click! (find (format "[widget-id=~a]" id)))` | Yes |
| `(bot:find selector)` | `(page-query-selector! (current-page) selector)` | Yes |

All actions operate on real browser DOM. `bot:find` returns an actual DOM element or `#f`.

### Concurrency

`raco congame simulate` (from `congame-cli/cli.rkt`) spawns N threads, each with its own Marionette connection on a separate port (`2829 + idx`). However, **Marionette is not thread-safe** (noted in `congame-tests/congame/bots.rkt` line 98-101). The test infrastructure uses a shared Marionette instance, which limits true concurrent testing.

### What `bot:autofill` Does

1. Reads JSON from a `<script data-autofill-options>` tag in the page (placed by `make-autofill-meta`)
2. Looks up the bot's `kind` in the JSON to get field values
3. For each field: finds the DOM element by name, types or clicks the value
4. Clicks submit

This is real browser interaction — if the form doesn't render, autofill fails.

## How Playwright Tests Work (from many-designs)

Joel's Playwright framework runs real Chromium instances (not Firefox) against a Docker server. Key differences from bots:

### Cross-participant coordination

```typescript
const results = await Promise.all(participants.map(runParticipant));
// After all complete, assert cross-participant properties:
const competitionCount = results.filter(r => r.isCompetition).length;
expect(competitionCount).toBe(2);
```

Bots run independently with no shared test scope. Playwright runs all participants in one test function with shared variables.

### Design-fidelity narratives

Each Playwright test starts with a 30-40 line block comment explaining the experimental design, chosen inputs, expected outcomes, and what each assertion verifies. Bot models have no equivalent — they're opaque `match` clauses.

### Content assertions

```typescript
const text = await getPageText(page);
expect(text).toContain('Your bonus: £0.70');
expect(text).not.toContain('opponent');
```

Bot models *could* do this (they have DOM access) but currently don't.

## Gap Analysis Revised

### Gaps that are convention/skill problems (fixable without changing congame)

1. **No content assertions in bot models** — bots have `bot:find` which returns DOM elements. A bot model could do `(define text (element-text (find "div.container")))` and assert on it. No existing study does this. The `create-study` and `study-review` skills should encourage/require it.

2. **Only 2 bot kinds per study** — most studies define `'liar` and `'honest`. More kinds (edge cases, ties, zero inputs) would improve coverage. Trivial to add.

3. **No structured documentation in bot models** — bot models are bare `match` expressions. Adding comments explaining what each step checks would improve readability but requires convention change, not code change.

### Gaps that are architectural (would require congame changes)

1. **Cross-participant assertions** — the bot framework runs each bot independently. There's no mechanism for bot A to check bot B's results page. `raco congame simulate` spawns threads but they don't coordinate. This would require a test harness wrapper around the bot framework.

2. **Marionette thread safety** — concurrent bots each need their own Marionette port. The test infrastructure's shared instance is a limitation for concurrent testing.

3. **Design document integration** — bots have no concept of "the design says X." Making bots design-aware would require a new abstraction layer (e.g., a design spec DSL that generates both the study and its test assertions). This is a research project, not a quick fix.

## Comparison Matrix (Revised)

| Capability | Bots (current) | Bots (with richer models) | Playwright |
|---|---|---|---|
| Study flow traversal | Yes | Yes | Yes |
| Form filling | Yes | Yes | Yes |
| Real browser DOM | Yes | Yes | Yes |
| Page content assertions | Not used | Yes (via Marionette) | Yes |
| Payment verification | Not used | Yes | Yes |
| Treatment isolation checks | Not used | Yes | Yes |
| Cross-participant assertions | No | Limited (no shared scope) | Yes |
| Design-fidelity narratives | No | No (convention gap) | Yes |
| Concurrent matchmaking | Partial (thread-safety issues) | Partial | Yes |
| Setup cost | Low (Racket only) | Low | Higher (Node, Docker) |
| Speed | Fast | Fast | Slower |

## Recommendations

1. **Enrich bot models immediately** — add `bot:find` + text assertions to bot models. This is low-hanging fruit: bots already have DOM access, they just don't use it for verification. Update `create-study` to generate bot models with content checks.

2. **Update study-review checklist** — add items: "bot model verifies payment amounts on results page," "bot model checks treatment-specific content appears/doesn't appear."

3. **Keep Playwright for integration tests** — cross-participant assertions and design-fidelity narratives remain Playwright's unique value. Use bots for fast per-participant smoke tests, Playwright for full integration tests before production.

4. **Consider a bot test harness** — a wrapper that runs N bots and collects their observations for cross-participant assertions. This would close the gap without requiring Playwright for most studies. Lower priority — Playwright already works.
