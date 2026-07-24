# The AI-Ready Ticket Template

This is the shape a UW ticket should take before it enters the Claude Code flow
(`/plan` → implement → `/validate` → `/review` → run locally → `/raise-pr`), described in
"Building with Claude Code on racing-fe-monorepo."

## What this template is — and is NOT

It exists for one reason: **`/plan` researches and plans from the ticket, so the ticket has
to actually say what the work is.** This template makes a ticket describe its work clearly
enough that `/plan` researches the *specifics* instead of guessing the *shape*.

The machine-readable "AI Implementation Contract" is produced by the repo's own
`/create-jira-ticket` command and parsed by `/plan` and `/raise-pr`; its schema lives with
those commands, not here. So the division of labour is:

- **This template + the skill** get a ticket to have **correct scope, a clear issue
  description, and (for bugs) real reproduction steps** — the human-owned inputs.
- **`/create-jira-ticket`** (run in the repo, by an engineer with the codebase) is what then
  generates the Implementation Contract from a well-described ticket, if the team wants it.

A ticket that is clearly scoped and described is exactly the input `/create-jira-ticket` and
`/plan` need. That is the whole goal here.

---

## Fields to set (native Jira fields, not just the description)

| Jira field | Rule |
| --- | --- |
| **Issue type** | `Story` (feature/enhancement), `Defect` (something is broken), `Spike` (investigate only, no fix), `Finding` (security report), `Task` (ops/config). Drives which description sections apply. |
| **Labels** | Include a **product** label (`FDR`, `TVG`, `4njbets`, `MEP`) and keep existing team/area labels (`Frontend`, `Backend`, `Mercenaries`, `MYSS`, …). |
| **Priority / Components / Assignee** | Left as triaged unless the user asks to change them. |

> **Why product is mandatory:** in this repo *product decides the codebase*. A UW bug on FDR
> vs TVG lives in a different place. The agent does **not** route by product — the
> engineer launches Claude in the right codebase — so the ticket must state the product. A
> multi-product change is always small, so it stays as one ticket covering all its products.

## Summary line

`[PRODUCT] <specific outcome>` — e.g. `[TVG] Truncate cut-off TVG Picks talent names`. If it
spans products, list them: `[FDR][TVG]`.

---

## Description structure

The description opens directly with the context (no section heading on the first block), then
has the sections for its work type. Keep it plain and readable.

### Opening context (all work types)

```
**What & where.** One or two sentences naming the exact feature/area and what's wrong or
wanted. Point at the specific screen/component, not a vague area.

**Scope — products & platforms.** Be explicit; don't leave it implied.
  - Products: FDR / TVG / 4njbets / MEP
  - Platforms per product: Desktop / Mobile Web / iOS / Android / RN / X-Sell
  - If multi-product: it's always a small change, so keep it as ONE ticket covering all the
    listed products.

**Out of scope.** What this ticket deliberately does not touch.

**Design (visual work only).** Figma link to the SPECIFIC node (not a whole-breakpoint
frame), and the target breakpoints (XS / S / M / L / XL). If it's visual and there's no
design, say so — /plan treats that as a blocker.
```

### Then, by work type — pick ONE

**Story / Task**
```
## Requirements / behaviour
- What the feature should do, in plain words. (Acceptance Criteria in Gherkin go in the
  shared section below.)

## Behind a flag? (if applicable)
- Flag/experiment key, and the expected behaviour BOTH on and off.
```

**Defect — clear description + reproduction**
```
## Issue description
- What the user sees vs. what they should see. Plain words — do NOT rely on a screenshot to
  carry the meaning; state it in text so /plan can read it.

## Steps to reproduce
1. …  2. …  3. …
- Expected result vs. actual result.
- Consistency: always / intermittent / only with specific data (name the data).
- Where seen: product + platform + device/browser (e.g. TVG, Mobile Web, iOS Safari).
- Who can reproduce: engineer / agent-can / needs-real-device. (Some bugs the agent can't
  reproduce — that doesn't make them unreal.)

## Developer: confirm mobile platform scope (mobile bugs only)
- A mobile bug reported on one platform is NOT assumed exclusive to it. The developer picking
  up the ticket should first try to reproduce on the product's OTHER mobile platform(s)
  (Mobile Web ↔ native iOS/Android ↔ RN) and expand the scope if it reproduces there too.
- Confirmed-on: <platforms the reporter verified>. To-check: <the other mobile platform(s)>.

## Screenshots
- Keep every image, and give EACH one a caption in the reporter's/user's own words saying what
  it shows and what to look at (e.g. "TVG: talent name 'Christina …' overflows its container
  and clips at the card edge, with no ellipsis"). An uncaptioned image is not a description —
  the person prepping the ticket describes each image (they can see it); the agent never
  captions from a guess.

## Known context (if any)
- Linked PRs, related tickets, a twin in the other product, anything from comments.
- Does the suspected area touch a SHARED component (urp-packages/DS)? If so, note which other
  products/consumers could be affected. (Do NOT pre-write a fix — cause stays for /debug.)
```

**Spike — investigate only**
```
## Question to answer
- The single question this spike resolves.
## Boundaries
- Timebox; deliverable is a findings write-up on the ticket, NOT a PR.
## Starting context
- Docs / dashboards / logs to start from.
```

**Finding — security report**
```
## Report provenance
- Source + link, date, reporter, weakness class.
## Affected assets / Impact / Repro
- Kept faithful to the original report.
## Remediation direction
- Intended fix direction; note if it touches shared infra.
```

### Acceptance Criteria — Gherkin (all work types)

Every ticket carries Acceptance Criteria that define "done", in the **Gherkin** pattern.
These go in an `## Acceptance Criteria` section **in the description** (not a custom field).

```
Scenario: <short name>
  Given <starting context>
  When <action>
  Then <observable, checkable outcome>
```

- Derive them from the confirmed scope, the issue description, and the reproduction steps.
- For a **Defect**, the fix's ACs are essentially "the reported reproduction no longer produces
  the bug" plus "no adjacent regression". Cover the happy path and the relevant edge/negative
  cases; for visual work, name the platform/breakpoint per scenario.
- **Mobile bugs:** include a scenario for confirming/holding the behaviour on the product's
  other mobile platform(s), paired with the developer reproduction task above.
- **Flags:** write scenarios for both on and off.
- These are **suggested** and approved by the ticket owner — don't assert an AC for behaviour
  nobody confirmed.

Example (for the TVG Picks talent-name defect):
```
Scenario: Long talent name is truncated to fit on TVG Mobile Web
  Given I am viewing TVG Picks on TVG Mobile Web
  And a pick has a talent name longer than its container
  Then the name is truncated to fit the container (ellipsis), matching FDR
  And no talent name overflows or is clipped mid-character

Scenario: Confirm whether the issue also affects TVG native mobile
  Given the same TVG Picks section on the TVG native app (iOS/Android)
  When a pick has a talent name longer than its container
  Then the developer records whether the overflow reproduces there
  And the ticket scope is expanded to that platform if it does
```

---

## Definition of Ready

A ticket is ready for the Claude Code flow when:

1. Summary is `[PRODUCT] <specific outcome>`.
2. The opening context is complete: what & where, the product/platform scope, out-of-scope,
   and (for visual work) a specific Figma node + breakpoints.
3. The work-type sections are filled — for a Defect that means a **text** issue description,
   **reproduction steps**, and a **caption for every image** (no meaning left trapped in an
   image). For a mobile bug, the **developer "reproduce on the other mobile platform(s)"
   task** is present.
4. **Acceptance Criteria are present in Gherkin** (as an `## Acceptance Criteria` section in
   the description), covering
   completion and — for mobile bugs — the cross-platform check.
5. Nothing critical is silently missing; anything genuinely unknown is called out as an open
   question for the team rather than guessed.

Meeting this bar is what lets `/plan` (and `/create-jira-ticket`, if the team runs it)
work from the ticket instead of around it.
