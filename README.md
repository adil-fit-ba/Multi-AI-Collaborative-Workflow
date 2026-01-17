# DIR Protocol: Multi‑AI Collaborative Workflow (Manual, Human‑Moderated)

**Version:** 3.2  
**Date:** January 2026  
**Repository:** https://github.com/adil-fit-ba/Multi-AI-Collaborative-Workflow/

---

## Who this document is for

This document is written for **humans**:
- team members participating in a multi‑LLM workflow (moderators, organizers, developers, reviewers),
- people who want to **apply** the protocol in practice,
- people who want to **test** it or provide **feedback**.

**LLM chats are not expected to read the rationale.** LLMs should be run using the **Onboarding Prompts** (Section 6), which are concise and instruction‑oriented.

<details>
<summary><strong>Why this separation (Humans vs LLMs)?</strong></summary>

LLMs tend to over‑generalize and “role drift” when given long narrative context. Humans benefit from understanding *why* rules exist; LLMs benefit from short, unambiguous directives. This protocol therefore keeps rationale **available** but **optional** for execution.
</details>

---

## 0) The gist (60 seconds)

- Treat each chat as a **new expert** (no assumed memory).
- Run **Dual Independent Review (DIR)**: two reviewers, blind to each other.
- Enforce **Evidence vs Hypothesis**: unverifiable claims must be labeled.
- Keep every chat on track using **Pinned Context** (always).
- Avoid parallel conflicting instructions using **hub‑and‑spoke task routing**.
- Resolve disagreements with a **Conflict Packet**, not “do you agree?”.

<details>
<summary><strong>Rationale</strong></summary>

These are the highest‑leverage guardrails observed in real multi‑chat work: preventing artifact confusion, minimizing conformity, and ensuring a single channel to the developer. The goal is reliability under context limits.
</details>

---

## 1) Core principles (non‑negotiable)

### 1.1 Dual Independent Review (DIR)
Every delivered artifact must be reviewed by **two independent reviewers** who do **not** see each other’s review until both are finished.

<details>
<summary><strong>Rationale</strong></summary>

Independent perspectives catch different classes of issues. “Second reviewer after seeing the first” tends to converge via conformity rather than critique, reducing error detection.
</details>

### 1.2 Evidence vs Hypothesis rule
Any reviewer claim must be tagged as:
- **Evidence**: verifiable (path/line, reproduction steps, logs, compile/test output), or
- **Hypothesis**: plausible but unverified.

<details>
<summary><strong>Rationale</strong></summary>

This blocks “hallucination propagation”: an unverified statement becoming downstream “truth”. It also makes conflicts resolvable by tests rather than authority.
</details>

### 1.3 Pinned Context discipline (every chat)
Every new chat starts with a compact **Pinned Context** block. No exceptions.

<details>
<summary><strong>Rationale</strong></summary>

Without forced restatement of current state, models (and humans) drift, mix artifacts, and answer outdated questions—especially when threads are long or repeated.
</details>

### 1.4 Hub‑and‑spoke task routing
The developer receives tasks from **one source only** (ORG; in Light mode MOD may temporarily act as ORG).

<details>
<summary><strong>Rationale</strong></summary>

Parallel tasking from multiple reviewers creates contradictory instructions and destroys traceability. A single routing point enables deduplication, prioritization, and consistent versioning.
</details>

---

## 2) Modes: Light vs Full

### 2.1 Light Mode (default)
Use for small/medium tasks. Roles:
- **MOD** (human) — controls the process, makes the final cut.
- **DEV** — implements.
- **REV‑A** and **REV‑B** — independent reviewers.

**Light Mode rule:** DIR is still mandatory.

<details>
<summary><strong>Rationale</strong></summary>

Even “small” changes can ship subtle breakage. Light mode removes administrative overhead while keeping the one mechanism that most reliably prevents misses: dual independent review.
</details>

### 2.2 Full Mode (escalation)
Use for complex tasks, repeated conflict, or high risk. Adds:
- **ORG** — project versioning + task routing + acceptance gate to DEV.
- **COORD** — coordinator who synthesizes disagreements and recommends a MOD decision.
- Optional: **TH‑A / TH‑B** — ideation specialists for architecture/research questions.

<details>
<summary><strong>Rationale</strong></summary>

Full mode exists to manage *coordination load*: deduplication, conflict resolution, and auditability when multiple moving parts exceed what the moderator can comfortably juggle ad‑hoc.
</details>

### 2.3 Escalation triggers (Light → Full)
Escalate if any of these hold:
- REV labels issue **High** or recommends **Block**,
- conflict not resolved after **one** structured conflict round,
- change touches public API / database schema / deployment pipeline,
- MOD feels loss of overview or repeated context confusion.

<details>
<summary><strong>Rationale</strong></summary>

Escalation is a safety valve. You only pay coordination overhead when risk is high or discussion becomes non‑productive.
</details>

---

## 3) Roles and responsibilities (what each role does)

### MOD (Moderator — human)
- Owns process, decides when to continue vs cut.
- Assigns DEC IDs and records final decisions.
- Approves what becomes “source of truth” in documentation.

**MOD is the only final decision‑maker.**

<details>
<summary><strong>Rationale</strong></summary>

A single accountable decision point prevents “committee paralysis” and keeps the workflow moving even when evidence is incomplete.
</details>

### ORG (Organizer)
- Maintains project versioning and artifact IDs.
- Converts reviews/conflicts into **task orders** for DEV.
- Ensures DEV receives a single, consistent instruction stream.
- Verifies deliverables include required packaging (MANIFEST, DELIVERY NOTE).

<details>
<summary><strong>Rationale</strong></summary>

ORG is the “routing + bookkeeping brain” so reviewers can focus on analysis and DEV can focus on implementation. It reduces cognitive load on MOD for sustained work.
</details>

### DEV (Developer / Implementer)
- Implements the **Task Order** exactly.
- Delivers artifact with **MANIFEST.md** + **DELIVERY NOTE**.
- Does not engage in strategy debate inside the DEV chat.

<details>
<summary><strong>Rationale</strong></summary>

Separating implementation from strategy prevents churn and keeps artifact‑bounded work efficient.
</details>

### REV‑A / REV‑B (Reviewers)
- Review the same artifact independently using the same brief.
- Produce a **REVIEW MEMO** with severity + recommendation.
- If disagreement exists, respond via **Conflict Packet** (no “agree?” prompts).

<details>
<summary><strong>Rationale</strong></summary>

Two reviewers with different reasoning styles catch complementary issues. The structured memo prevents reviews from becoming vague opinions.
</details>

### COORD (Coordinator) — one role, two modes
COORD does **not** implement and does **not** task DEV directly.
COORD’s job is to reduce ambiguity and help MOD decide.

**COORD‑IDEA (Idea mode):**  
- merges competing ideas into 2–3 options,
- lists assumptions and trade‑offs,
- recommends an option to MOD.

**COORD‑DISPUTE (Dispute mode):**  
- resolves contradictions (what is correct *for this artifact*),
- demands evidence or proposes a test/spike,
- produces a recommendation to MOD (accept A/B, run test, MOD cut).

<details>
<summary><strong>Rationale</strong></summary>

Using one coordinator with two modes avoids role explosion (REV‑COORD vs THINK‑COORD) while preserving the same disciplined mechanism: structured convergence + MOD decision.
</details>

### TH‑A / TH‑B (Thinkers — optional)
Use when you need architecture/research exploration before implementation.
- Propose approaches, risks, long‑term implications.
- Do **not** write production code in thinker chats.

<details>
<summary><strong>Rationale</strong></summary>

Thinker chats are useful for high‑level exploration, but they easily drift into pseudo‑implementation. Keeping them non‑coding protects clarity and handoff quality.
</details>

---

## 4) Artifacts, versioning, and deliverables

### 4.1 Artifact naming (example)
Use a unique artifact ID and version:
- `project_v29.4.3.zip`
- `project_v29.4.4_patch1.zip`

ORG owns versioning.

<details>
<summary><strong>Rationale</strong></summary>

Unique names prevent “final_FINAL.zip” chaos and make reviews traceable.
</details>

### 4.2 Required packaging for any delivered artifact
DEV delivers:
1) the artifact (zip/repo link/etc.)
2) **MANIFEST.md** (what’s inside, key diffs, versions)
3) **DELIVERY NOTE** (how to verify, what changed, constraints respected)

<details>
<summary><strong>Rationale</strong></summary>

Without standardized packaging, reviewers waste time reconstructing context, and mistakes slip through (wrong branch, wrong file set, missing migrations, etc.).
</details>

### 4.3 Rollback (when a delivery is rejected)
- Never delete history. Mark artifact as **Rejected**.
- ORG issues a new task order: “revert to X” or “fix Y”.
- DEV delivers a new version (`+patch`).

<details>
<summary><strong>Rationale</strong></summary>

Rejecting without losing traceability enables audit trails and reduces repeated debate about “what happened”.
</details>

---

## 5) Conflict handling (anti‑ping‑pong)

### 5.1 What NOT to do
Do not ask a reviewer: **“Do you agree with reviewer X?”**

Instead: create a **Conflict Packet** that asks for:
- what claim is wrong and why,
- what evidence would decide it,
- what minimal test/spike can settle it.

<details>
<summary><strong>Rationale</strong></summary>

Agreement prompts trigger conformity. Conflict Packets trigger falsifiable reasoning and concrete next steps.
</details>

### 5.2 Conflict workflow
1) REV‑A and REV‑B complete blind reviews.
2) If disagreement: MOD/ORG writes Conflict Packet.
3) Each reviewer responds once with evidence/test suggestions.
4) COORD‑DISPUTE may synthesize if needed.
5) MOD makes final cut; ORG turns it into a task order if changes are required.

<details>
<summary><strong>Rationale</strong></summary>

A bounded round count prevents endless ping‑pong while still extracting the key information needed for a decision.
</details>

---

## 6) Onboarding prompts (copy/paste)

> **Rule:** LLM chats should follow the onboarding prompt and **ignore rationale**.  
> Only the short “Execution” parts matter for LLM operation.

### 6.1 PINNED CONTEXT (General)
```text
PINNED CONTEXT (General)
Protocol: DIR Protocol v3.2
Role: [MOD / ORG / DEV / REV-A / REV-B / COORD / TH-A / TH-B]
Artifact: [artifact-id]
Goal: [1 sentence]
Out of scope: [2–3 bullets]
Decisions already made: [up to 2 bullets or DEC-IDs]
Open questions: [max 3]
Constraints: [max 5]
```

### 6.2 PINNED CONTEXT (Review)
```text
PINNED CONTEXT (Review)
Artifact: [artifact-id]
Review goal: [1 sentence]
Out of scope: [2–3 bullets]
Already decided: [DEC-IDs or 2 bullets]
Open items: [max 3]
IMPORTANT: Answer only about this artifact.
```

### 6.3 Onboarding — DEV
```text
You are DEV (Implementer). You only implement the Task Order below.
Do not debate strategy. Do not change scope.
Output: DELIVERY NOTE + MANIFEST.md summary.

[PINNED CONTEXT (General)]

TASK ORDER:
[paste]
```

### 6.4 Onboarding — Reviewer (REV-A / REV-B)
```text
You are an independent reviewer. You must not assume continuity from other chats.
You must label claims as Evidence vs Hypothesis.
You must not seek agreement—focus on falsifiable critique.
Output format: REVIEW MEMO.

[PINNED CONTEXT (Review)]

ARTIFACT / DIFF / CONTEXT:
[paste]
```

### 6.5 Onboarding — COORD (Idea mode)
```text
You are COORD in IDEA mode. Your job is to synthesize competing proposals into options.
Do not implement. Do not task DEV.
Output: 2–3 options + trade-offs + recommended option to MOD.

[PINNED CONTEXT (General)]

INPUTS (proposal A, proposal B, constraints):
[paste]
```

### 6.6 Onboarding — COORD (Dispute mode)
```text
You are COORD in DISPUTE mode. Your job is to resolve contradictions about correctness.
Demand Evidence; if missing, propose the smallest test/spike.
Do not implement. Do not task DEV.
Output: Recommendation to MOD (accept A/B, run test, MOD cut) + rationale.

[PINNED CONTEXT (General)]

CONFLICT PACKET:
[paste]
```

### 6.7 Onboarding — ORG
```text
You are ORG. You own versioning and task routing.
You convert reviews/decisions into a single Task Order for DEV.
You ensure deliverables include MANIFEST + DELIVERY NOTE.

[PINNED CONTEXT (General)]

INPUTS:
[reviews, decisions, constraints]
```

---

## 7) Templates (execution outputs)

### 7.1 REVIEW MEMO (REV)
```text
REVIEW MEMO
Artifact: [id]
Severity: Low / Medium / High
Recommendation: Merge / Merge with fixes / Block

Findings:
- [Evidence|Hypothesis] ...
- ...

Stop-ship issues (if any):
- ...

Suggested tests / verification:
- ...

Verified by: compile (yes/no), run (yes/no), tests (yes/no)
How verified: ...
```

### 7.2 CONFLICT PACKET (MOD/ORG)
```text
CONFLICT PACKET
Artifact: [id]
Question to resolve: [1 sentence]

Claim A (from REV-A):
- ...

Claim B (from REV-B):
- ...

What evidence would decide this?
- ...

Smallest test/spike to settle it:
- ...

Constraints:
- ...
```

### 7.3 TASK ORDER (ORG → DEV)
```text
TASK ORDER
Artifact target: [new artifact id/version]

Goal:
- ...

Scope (do):
- ...

Out of scope (do not touch):
- ...

Acceptance criteria:
- [ ]
- [ ]

Constraints:
- ...

Verification steps:
- ...
```

### 7.4 DELIVERY NOTE (DEV)
```text
DELIVERY NOTE
Artifact: [id]
Summary of changes:
- ...

Files/modules touched:
- ...

How to verify:
- ...

Known limitations / assumptions:
- ...
```

### 7.5 MANIFEST.md (DEV)
```text
MANIFEST
Artifact: [id]
Contents:
- ...

Key diffs:
- ...

Dependencies / versions:
- ...

Build/run notes:
- ...
```

---

## 8) Feedback and contribution

If you test this protocol or want to suggest improvements:
- open an issue with: **context + what failed + what you changed + why**
- include the smallest reproducible example (artifact id + pinned context)

<details>
<summary><strong>Rationale</strong></summary>

Clear feedback structure enables iteration without turning the repository into a chat log. Reproducibility is the only scalable way to improve a protocol.
</details>

---

## Changelog

- **3.2 (Jan 2026):** Introduced **COORD** as a single role with two modes (IDEA/DISPUTE); clarified ORG versioning responsibilities; strengthened “humans vs LLMs” separation via rationale blocks; kept execution prompts concise for LLMs.
