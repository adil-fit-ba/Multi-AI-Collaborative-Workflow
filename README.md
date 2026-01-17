# Multi‑AI Collaborative Workflow (Email Team)

A system for orchestrating multiple AI models on complex intellectual tasks, designed as a **team of experts communicating 1‑on‑1 (like email)**.

**Status:** Source of truth (single file)  
**Version:** 2.5 (EN)  
**Date:** January 2026  
**Current models:** Claude Opus 4.5, Claude Sonnet 4.5, GPT‑5.2, Gemini Pro 3

---

## 0) The gist (60 seconds)

- **Roles matter more than models.** Models are replaceable; responsibility is not.
- **Every chat is a new person.** Assume knowledge = 0 → Pinned Context is mandatory.
- **The user (MOD) is the process moderator.** You cut the tempo, decide, and edit what enters the documentation.
- **Dual review is REQUIRED even for small tasks.** (GPT + Opus, same prompt, same artifact)
- **DEV receives tasks from a single channel only** (ORG, or MOD acting as ORG) → no parallel/conflicting instructions.

---

## 0.1 Why this way (Design Rationale)

These aren’t “rules for rules’ sake” — they prevent failure modes that repeatedly happen when multiple chats work in parallel.

| Rule | Why |
|---|---|
| **Roles > models** | Models change; responsibilities must not. Otherwise you lose auditability and process control. |
| **Every chat = a new person** | Prevents the “continuity illusion”; forces minimal context and reduces hallucinations. |
| **Hub‑and‑spoke (DEV receives tasks from one channel only)** | The most effective way to avoid conflicting instructions and “patch‑on‑patch” chaos. |
| **Dual review always (GPT + Opus)** | Different reasoning styles catch different bugs; cost is small, payoff is big. |
| **Don’t show reviewers each other’s review before they finish** | Preserves independence; otherwise you get conformity instead of a second perspective. |
| **Evidence > opinion (no evidence = Hypothesis)** | Prevents intuition and “product opinion” from being treated as fact. |
| **Resolve conflict by test or cut, not ping‑pong** | Saves context and time; debate without new evidence is noise. |
| **REV‑COORD = Opus 4.5 in Full mode** | A coordinator needs stability and factual consistency, not “far‑future implications”. |
| **Pinned Context + GPT guardrail** | Direct antidote to artifact mix‑ups in long chats (especially during reviews). |
| **Artifact naming + MANIFEST + Rollback** | Reproducibility + forensic trail: what shipped, when, how verified, and how to revert. |
| **Light vs Full** | Minimizes overhead when risk is low; escalates only when truly needed (High/Block, public API, loss of overview). |

---

## 0.2 Quick Start (Light mode in 5 steps)

For those who want to start immediately:

```
1. Write a TASK ORDER → send to DEV
2. DEV returns DELIVERY NOTE + artifact + MANIFEST
3. Send the same artifact to REV-G and REV-C (same prompt)
4. If they agree → fix backlog → DEV
5. If they disagree → Conflict Packet → max 1 round → cut
```

Details are in the sections below.

---

## 1) Models (without roles)

| Model | Strengths | Weaknesses / risks |
|---|---|---|
| **GPT‑5.2** | Best for structuring, deduping, prioritizing, systematic review, methodology, task breakdown | Can drift into too much process; weaker as a pure implementer; needs explicit “don’t code” |
| **Claude Opus 4.5** | Deep argumentation, debate, alternatives, synthesis; good product sense | Context limit → needs Pinned Context; sometimes less checklist‑systematic |
| **Claude Sonnet 4.5** | Best “workhorse” for implementation and delivery; stable execution of instructions | If the brief is vague, it fills gaps with assumptions → requires Task Order + acceptance criteria |
| **Gemini Pro 3** | Huge context; excellent as archivist/minutes‑taker and long‑history synthesis | Can be generic → needs templates and curated input, not “all messages” |

---

## 2) Work roles → recommended model

| Role | Tag | Recommended model | Comment / guardrail |
|---|---|---|---|
| Moderator | **[MOD]** | You | Cuts, priorities, decisions, final merge into documentation. |
| Organizer / Dispatcher (hub) | **[ORG]** | **GPT‑5.2** | Receives inputs, dedupes, produces **Task Orders**. Does not go deep into code. **Does not code.** |
| Thinker (strategy/methodology) | **[TH]** | GPT‑5.2 + Opus 4.5 (parallel) | Same brief goes to both (different reasoning style). **They do not code.** |
| Reviewer A | **[REV‑G]** | **GPT‑5.2** | Independent review. Output = **REVIEW MEMO** (not tasks). |
| Reviewer B | **[REV‑C]** | **Opus 4.5** | Independent review; often catches different failures. Output = REVIEW MEMO. |
| Implementer | **[DEV]** | **Sonnet 4.5** (Opus 4.5 for heavy refactors) | Works only from Task Orders. Allowed 1–3 short questions when assumptions block. |
| Recorder / Archivist (optional) | **[LOG]** | **Gemini Pro 3** | Receives curated LOG ENTRY + MANIFEST summaries. |
| Conflict coordinator (optional) | **[REV‑COORD]** | **Opus 4.5** (Full) / MOD (Light) | **Progress moderator**: demands evidence, cuts ping‑pong, returns a Coord Report. In Light mode MOD does this (no separate chat). |

**Instance rule:** one role = one chat. If you need parallelism: `[REV‑G2]`, `[DEV‑2]`… (each is a separate chat).

**Note on TH:** Use TH only when you need strategic/methodological discussion before implementation (e.g., architecture, major trade‑offs). For pure bugfixes and small features, skip TH.

---

## 3) Two operating modes

### 3.1 Light (default for small tasks)

Minimal, but reliable:
- **MOD** (you) + **DEV** + **REV‑G & REV‑C**
- **ORG** is optional (if absent, MOD “acts as ORG” and remains the **single source of tasks**)
- **Dual review is required**
- **No separate conflict‑coordinator chat**: MOD cuts (max 1 conflict round in Light)

*Why Light: minimal friction, while keeping the two highest‑leverage protections (single task channel + dual review).* 

### 3.2 Full (when it escalates)

**Escalate Light → Full (checklist):**
- [ ] A reviewer returns **High** or **Block**
- [ ] The conflict is not resolved in **1 round** (Light limit)
- [ ] The change touches **>3 files** or a **public API**
- [ ] You (MOD) feel you’re losing overview / risk is growing

In Full mode you add ORG (if not already) and, when needed, `[REV‑COORD]` as a separate chat (default: **Opus 4.5 as progress moderator**).

*Why Full: increased overhead only when risk is real (High/Block, multiple modules, public API, or a conflict that needs proof/tests).* 

---

## 4) Communication topology (email‑only, no meetings)

**Golden rule:** DEV never receives parallel instructions from multiple chats.

```
REV/TH → ORG (or MOD)     inputs go into the hub
ORG/MOD → DEV              tasks from a single channel only
DEV → ORG/MOD              Delivery Note + artifact + MANIFEST
ORG/MOD → REV‑G and REV‑C  same artifact, same review brief
ORG/MOD → REV‑COORD        only for hard conflicts (Full mode)
```

---

## 5) Non‑negotiable rules

1. **Every chat is a new person.** Put Pinned Context at the top.
2. **Dual review (DIR) is mandatory** even for small tasks.
3. **Do not show reviewers each other’s reviews before both finish.**
4. **If there’s no evidence → the status is Hypothesis.**
5. **Do not ask “do you agree?”** When there is conflict, request evidence and a falsification test.

### 5.1 Exception: verification without conformity

If you suspect GPT mixed artifacts, you may send a **curated list of items** (not the full review) and ask for labels:
- **VALID** — confirmed
- **INVALID** — incorrect
- **UNCERTAIN** — needs verification

For each label, ask for evidence/test where possible. This is not “seeking agreement” — it’s fact verification.

### 5.2 GPT guardrail (file mix‑ups / long context)

When you notice GPT mixing artifacts or adding facts not present in the current input:

- **Don’t feed the full history.**
- Send only: **Artifact ID + Pinned Context (5 bullets) + your question**.
- Insist explicitly: **“Answer only for this artifact.”**

---

## 6) Standard workflow (Light)

```
1) MOD/ORG → DEV:        Task Order (scope + acceptance + constraints)
2) DEV → MOD/ORG:        Delivery Note + artifact + MANIFEST
3) MOD/ORG → REV‑G/C:    same review task, same artifact
4) If they agree:         fix backlog → DEV
5) If they disagree:      Conflict Packet → max 1 round → cut or spike/test
```

**Progress** exists only if something new appears: evidence, a minimal test, narrowed dispute, or a changed stance.

---

## 6.5 Rollback / Abort (when a delivery is not acceptable)

**Triggers:**
- A reviewer returns **Block** with a STOP‑SHIP issue
- DEV reports the delivery is fundamentally wrong
- MOD discovers a critical problem after merge

**Procedure:**

1. MOD marks the artifact as **rejected** (do not delete — keep audit trail)
2. MOD/ORG creates a new **Task Order** with clear: `revert to <X>` or `fix <Y>`
3. DEV delivers a **patch** (vMAJOR.MINOR.PATCH+1) with explanation in DELIVERY NOTE + MANIFEST

*Why: artifacts aren’t deleted to preserve audit trail; patch versions provide reproducibility and a clear attempt history.*

---

## 7) Reviewer conflicts (Light: MOD cuts)

### 7.1 Conflict Packet (copy/paste)

```text
I have a conflict between two reviews. I need your reply in 5–10 lines:

A-claim (other reviewer): <1 sentence>
B-claim (yours): <1 sentence>

1) Which claim is more correct and WHY?
2) Evidence (path:line / repro / log). If you have no evidence, label it "hypothesis".
3) One minimal test that would falsify the wrong claim.
4) Confidence 0–100%.
```

### 7.2 Conflict rules

- If both provide **evidence** and still disagree → **mini spike/test** is the default
- If the dispute is a “judgement call” (UX, trade‑off) → MOD cuts and moves on
- If the debate repeats without progress → **cut immediately** (don’t waste context)

### 7.3 When to enable a separate `[REV‑COORD]`

Only in Full mode, when it’s High/Block or the dispute is hard and persists.

---

## 8) Pinned Context

### 8.1 PINNED CONTEXT (general)

For TH, DEV, ORG and new chats:

```text
CURRENT STATE:
• Topic:
• Goal of this session:
• Last decision (if any):
• Open questions (max 3):
• Constraints (e.g., no migrations, no breaking API):
• Input reference (link/commit/artifact):
```

### 8.2 PINNED CONTEXT (review)

Short version for REV‑G/REV‑C (especially when GPT is confused):

```text
• Artifact: <name_version.zip>
• Review goal: <1 sentence>
• Out of scope: <2–3 items>
• Already decided: <DEC-### or 2 bullets>
• Open items: <max 3>
```

---

## 9) Templates (copy/paste)

### 9.1 REVIEW MEMO (for REV‑G and REV‑C)

```text
[REV] REVIEW MEMO
Target: <artifact / branch / commit>

Risk class: Low | Medium | High
Recommendation: Merge | Merge with fixes | Block

Findings:
- ISSUE-001 | Severity: Block/High/Med/Low | Claim | Evidence | Proposed fix

Risk notes:
- ...

Quick wins (max 5):
- ...

Questions (max 3):
- ...

Verified by: compile / run / tests (yes/no)
How verified: ...
```

*Note: “Verified by” is at the end because you read findings first, then check how it was verified.*

### 9.2 TASK ORDER (MOD/ORG → DEV)

```text
[ORG] TASK ORDER
Goal: <1 sentence>

Tasks:
- TASK-001: ...
  Acceptance:
  - [ ] ...
  Constraints:
  - ...

Out of scope:
- ...

Test/Verification:
- ...
```

### 9.3 DELIVERY NOTE (DEV → MOD/ORG)

> **Rule:** DEV always delivers **artifact + MANIFEST.md + DELIVERY NOTE** together.

```text
[DEV] DELIVERY NOTE
Artifact: <name + version>

Completed:
- TASK-001

Blocked/Not done:
- ...

Assumptions:
- ...

How to verify:
1) ...
2) ...
```

---

## 10) LOG ENTRY (if you use Gemini)

Gemini records only **curated** entries (not every message):

```text
[LOG ENTRY]
Timestamp: YYYY-MM-DD HH:MM
Actor: [ORG] | [TH] | [REV-G] | [REV-C] | [DEV] | [MOD]
Topic: <short>
Type: Idea | Decision | OpenQuestion | Delivery | Review | Conflict | Patch
Ref: DEC-### / ISSUE-### / TASK-### (if any)

Content:
- ...

Next:
- ...
```

---

## 11) Artifact naming + MANIFEST (mandatory for deliveries)

> **Rule:** MANIFEST.md is written by **DEV** for every delivery (Gemini/LOG stores only the textual summary, not the binary).

### 11.1 Filename format

```text
<Project>_vMAJOR.MINOR.PATCH_YYYYMMDD-HHMM_<ActorTag>.zip
```

Example: `MultiCriteriaAgent_v29.4.3_20260117-1042_DEV.zip`

### 11.2 MANIFEST.md (in the zip root)

```markdown
# MANIFEST

**Project:** <name>
**Version:** v#.#.#
**Date:** YYYY-MM-DD HH:MM
**Actor:** [DEV]

## Changelog (5–10 bullets)
- 

## Files changed
- `path/to/file.ext` — short note

## How to run / verify
1.
2.

## Known limitations
- 

## Related tasks/issues
- TASK-###
- ISSUE-###
```

---

## 12) Minimal documentation discipline

- Tags are mandatory only for things you copy into LOG or into this document
- Decisions and cuts should be short (3–7 bullets) and include “what” + “why”
- If something is later superseded, write “superseded” and keep the trace (don’t delete history)

**Numbering:**
- **ISSUE-###** is created by a reviewer (in REVIEW MEMO)
- **TASK-###** is created by ORG/MOD (in TASK ORDER)
- **DEC-###** is created by MOD (for decisions/cuts)
- Relationships are recorded in MANIFEST (“Related tasks/issues”)

---

## 13) Cheat Sheet

### Tags

```
[MOD]        Moderator (you)
[ORG]        Organizer/Dispatcher
[TH]         Thinker (strategy)
[REV-G]      Reviewer A (GPT)
[REV-C]      Reviewer B (Claude)
[DEV]        Implementer
[LOG]        Recorder (Gemini)
[REV-COORD]  Conflict coordinator (Full)
```

### Workflow (Light)

```
TASK ORDER → DEV → DELIVERY + MANIFEST → REV-G + REV-C → fix/merge or Conflict Packet
```

### Review Risk/Recommendation

```
Risk:           Low | Medium | High
Recommendation: Merge | Merge with fixes | Block
```

### Escalation Light → Full

```
[ ] High/Block finding
[ ] Conflict not resolved in 1 round
[ ] >3 files or public API
[ ] You’re losing overview
```

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 2.5 | Jan 2026 | Quick Start, Cheat Sheet, ISSUE/TASK/DEC numbering, Rollback trigger, PINNED CONTEXT split (general/review), “Verified by” moved to end of REVIEW MEMO |
| 2.4 | Jan 2026 | Design Rationale (0.1), Rollback procedure, escalation checklist |
| 2.1 | Jan 2026 | Light/Full modes, Conflict Packet |
| 2.0 | Jan 2026 | Hub‑and‑spoke, Dual Independent Review (DIR) |
