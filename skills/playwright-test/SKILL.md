---
name: playwright-test
description: |
  Write a Playwright browser test for a congame study. Use when the user
  wants to create an automated end-to-end test that runs participants
  through a study on the Docker server.
argument-hint: "[study-file]"
---

Write a Playwright test for a congame study. The test runs multiple
browser participants concurrently against the Docker server. Tests
should verify both that the study code runs without errors AND that
the study faithfully implements the original experimental design.

Before starting, read the study source file, the corresponding design
document, and the existing test helpers in `tests/helpers/`.

## 1. Identify the study

- If a file path is provided as `$ARGUMENTS`, use that.
- Otherwise, check conversation context for a recently discussed study.
- If still unclear, ask the user which study to test.

Read the full study `.rkt` file before proceeding.

## 2. Read the design document

Look for the study's design document at `study designs/{STUDY-ID}.md`
(e.g., `study designs/SBL89.md`). Read the full document. Extract:

- **Task description:** What participants actually do (e.g., report a
  number, make an investment, answer questions)
- **Treatment conditions:** Exactly how competition and control differ
- **Payment structure:** Show-up fee, bonus formulas, who gets paid
  what and when
- **Outcome variable:** What the experiment measures (e.g., proportion
  of "yes" reports, amount claimed vs. actual)
- **Group structure:** How many per group, roles, matching rules
- **Information timing:** What participants should/shouldn't know and
  when (e.g., "rank revealed only after all finish")

If the design document doesn't exist, ask the user to provide context
about the study's intended design.

## 3. Analyze the study flow

Trace the `defstudy` graph to determine:

- **Total participants needed:** Count how many participants are required
  for all matchmaking groups to fill. Use `2 × group-size` for studies
  with treatment/control branching where only treatment enters the
  matchmaker (e.g., `make-matchmaker 2` with only treatment matched →
  need 4 total for 2 treatment + 2 control). For studies where all
  participants enter the matchmaker, use just the group size.
- **Treatment detection:** Find a text string unique to one treatment's
  instructions page that can distinguish it from the other. Read the
  actual instruction step bodies to pick the string.
- **Step sequence for each path:** List every step a participant visits,
  noting for each:
  - `click` — page has a `@button{...}` → use `clickButton(page, label)`
  - `form` — page has a form → use `selectRadio`/`fillNumber`/
    `selectDropdown` + `submitForm`
  - `skip` — step calls `(skip)` → invisible, browser follows redirect
  - `wait` — page has `@refresh-every[...]` → use `waitForAdvance(page)`
- **Form field values:** For each form, determine valid input values
  from the step code. Radio values are the first element of each pair
  in the choices list (e.g., `("3" . "80 pence")` → value `"3"`).
  Number inputs need a value within `min`/`max` range. Select dropdowns
  need a valid option value.

## 4. Write the test

Create `tests/studies/{study-name}.spec.ts`.

### Narrative header

Start the file with a block comment explaining the test in plain
language. This comment should be readable by someone unfamiliar with
the code and should cover:

- What the study measures experimentally (from the design document)
- How the two treatments differ
- What specific input values the test uses and why
- What outcomes are expected given those inputs (e.g., "participant 0
  reports 70, participant 1 reports 30, so participant 0 should win")
- What each assertion is checking and why it matters for design fidelity

This narrative is the most important part of the test file — it turns
the test from "does the code crash?" into "does the study work as the
researchers intended?"

### Template

Follow this template after the narrative comment:

```typescript
import { test, expect } from '@playwright/test';
import {
  clickButton,
  submitForm,
  fillNumber,
  selectRadio,
  selectDropdown,
  getPageText,
  createParticipants,
  waitForAdvance,
  cleanupParticipants,
  type Participant,
} from '../helpers';

const SLUG = process.env.STUDY_SLUG || '{default-slug}';

async function runParticipant(p: Participant): Promise<{ isCompetition: boolean }> {
  const { page, id } = p;

  // ... step-by-step navigation ...

  return { isCompetition };
}

test('{STUDY}: N participants complete study', async ({ browser }) => {
  const participants = await createParticipants(browser, N, SLUG);
  try {
    const results = await Promise.all(participants.map(runParticipant));
    // ... assertions ...
  } finally {
    await cleanupParticipants(participants);
  }
});
```

### Key patterns

**Treatment detection** — after the welcome click, `(skip)` steps
redirect through to the instructions page. Read page text to detect
which treatment path:

```typescript
const text = await getPageText(page);
const isCompetition = text.includes('unique competition phrase');
```

**Matchmaking waits** — for each `@refresh-every` step, call
`waitForAdvance`. Chain multiple calls if there are consecutive wait
steps (e.g., matchmaking wait → opponent score wait):

```typescript
await waitForAdvance(page, 90_000);  // matchmaking
await waitForAdvance(page, 30_000);  // opponent score
```

**Form values per participant** — spread values across participants
using `p.id` so different participants submit different data:

```typescript
await fillNumber(page, 30 + p.id * 20);  // 30, 50, 70, 90
```

**Treatment balance assertion** — for balanced-shuffle studies:

```typescript
const competitionCount = results.filter(r => r.isCompetition).length;
expect(competitionCount).toBe(expectedCount);
```

### Admin interface testing

Some studies have admin-only steps (e.g., triggering score computation,
advancing phases). These use `current-participant-owner?` to route the
study owner to an admin interface while regular participants see the
normal study flow.

To test admin functionality:

1. Run participants through the study first (using `createParticipants`)
2. Create an admin session with `createAdminParticipant(browser, slug)`
3. Read admin page content with `getLoggedInPageText(page)` (NOT
   `getPageText` — see note below)
4. Interact with admin buttons using `clickButton`
5. Have participants reload to verify post-admin changes

```typescript
// After participants complete the study...
const admin = await createAdminParticipant(browser, SLUG);
try {
  const adminText = await getLoggedInPageText(admin.page);
  expect(adminText).toContain('Admin Interface');

  await clickButton(admin.page, 'Compute Scores');

  const postText = await getLoggedInPageText(admin.page);
  expect(postText).toContain('Scored');
} finally {
  await admin.context.close();
}
```

**Critical: `getPageText` vs `getLoggedInPageText`** — Logged-in
congame pages have two `div.container` elements: the first is the
navigation bar, the second is the study content. `getPageText` grabs
`.first()` (the nav), so it returns text like "DashboardLog outAdmin".
Use `getLoggedInPageText` for logged-in pages — it grabs `.last()`
to get the actual study content. Anonymous participant pages only have
one `div.container`, so `getPageText` works fine for them.

**Admin enrollment** — The admin must be enrolled in the study instance
before they can access it. `createAdminParticipant` handles this by
visiting `/dashboard`, finding the study by slug, and clicking
Enroll/Resume. The admin credentials are for the local Docker server
(`admin@congame.local` / `admin`, created by `congame-web/local.rkt`).

## 5. Design-based assertions

Beyond verifying the study runs without errors, derive assertions from
the design document that verify the study implements the experiment
correctly. For each category below, add `expect()` calls at the
appropriate point in the test.

### Payment correctness

The test chooses specific input values, so expected payments are
deterministic. Compute them from the design's payment formulas and
assert the results page shows matching amounts. For example, if a
participant reports 70 and the bonus is `number * £0.01`, the results
page should show `£0.70`.

### Treatment-specific content

Assert that competition participants see competition-specific text
(e.g., opponent scores, ranking, "the other participant") and control
participants see control-specific text. Also assert the **absence** of
cross-treatment content (e.g., control should NOT see "opponent" or
"rank").

### Outcome logic

If the design specifies who wins (e.g., higher number wins), verify
the results page correctly identifies the winner and loser given the
test's known inputs. If there's a tiebreaker rule, consider testing
a tie scenario.

### Information timing

If the design says participants shouldn't learn certain information
until later (e.g., "rank revealed only after all participants finish"),
assert that intermediate pages do NOT contain that information. Use
`expect(text).not.toContain(...)`.

### Group structure

Verify the correct number of participants are matched (e.g., pairs
vs groups of 4). For matchmaking studies, the test's participant count
and the assertions on group results implicitly test this — make it
explicit with a comment.

### What NOT to assert

Don't assert on randomized outcomes the test can't predict (e.g., a
coin flip result, a shuffled order). Focus on deterministic logic that
follows from the test's chosen inputs.

## 6. Congame HTML behavior (critical)

These are hard-won lessons from building the test framework. Do NOT
deviate from these patterns:

### Unpoly AJAX interception

Congame buttons render as
`<a class="button next-button" up-follow=".step">`. The `up-follow`
attribute causes the Unpoly framework to intercept clicks and do AJAX
partial page replacement instead of full navigation. **This breaks
Playwright's navigation detection.**

**Fix:** `clickButton` reads the button's `href` and navigates via
`page.goto(href)` instead of clicking. This is already implemented in
the helpers. Always use `clickButton`, never `locator.click()` on
navigation buttons.

### Form submission is NOT intercepted

Forms (`<form method="POST">`) do NOT have Unpoly attributes, so
`submitForm` uses a normal `locator.click()` on the submit button.
This triggers a standard form POST + full page navigation.

### Anonymous login redirect page

`/_anon-login/{slug}` does NOT redirect automatically. It renders a
"Redirecting..." page with a "click here" link. The `anonLogin` helper
handles this by navigating to `/study/{slug}` after hitting the login
endpoint. The session cookie persists across both requests.

### Waiting pages use setTimeout

`@refresh-every[N]` renders as
`setTimeout(() => document.location.reload(), N*1000)`. This is real
navigation that Playwright can detect. `waitForAdvance` polls by
checking whether any `<script>` tag contains `document.location.reload`.

### Skip steps are invisible

Steps that call `(skip)` return a server redirect. The browser follows
the redirect chain transparently. After a button click or form submit,
you may pass through multiple skip steps before landing on a visible
page.

## 7. Running the test

The study must be uploaded and have an active instance on the Docker
server before running.

```bash
cd tests
STUDY_SLUG={slug} npx playwright test studies/{study-name}.spec.ts
```

For visual debugging: `npx playwright test --headed`

Default server URL is `http://127.0.0.1:5100`. Override with
`SERVER_URL` env var.

## 8. Reference: available helpers

All imported from `../helpers`:

| Function | Purpose |
|---|---|
| `anonLogin(page, slug)` | Create anonymous participant, navigate to study |
| `clickButton(page, label?)` | Click `@button{...}` via href (bypasses Unpoly) |
| `submitForm(page)` | Click form submit button |
| `fillNumber(page, value, nth?)` | Fill `input[type=number]` |
| `selectRadio(page, value)` | Click `input[type=radio]` by value |
| `selectDropdown(page, value, nth?)` | Select `<option>` by value |
| `getPageText(page)` | Get text content of `div.container` (anon pages) |
| `pageContains(page, text)` | Boolean text check |
| `createParticipants(browser, count, slug)` | Create N isolated browser contexts |
| `waitForAdvance(page, timeout?)` | Wait for matchmaking/refresh page to advance |
| `cleanupParticipants(participants)` | Close all browser contexts |
| `adminLogin(page)` | Log in as the Docker admin user (`admin@congame.local`) |
| `adminEnrollInStudy(page, slug)` | Enroll admin in a study via `/dashboard` |
| `createAdminParticipant(browser, slug)` | Login + enroll in one call; returns `{ context, page }` |
| `getLoggedInPageText(page)` | Get study content from logged-in page (last `div.container`) |

## 9. Working examples

- `tests/studies/sbl89.spec.ts` — 4 participants, treatment branching,
  matchmaking, and opponent score exchange.
- `tests/studies/gwp43.spec.ts` — 4 participants, admin-triggered
  scoring, post-completion results page. Demonstrates
  `createAdminParticipant` and `getLoggedInPageText` helpers.
