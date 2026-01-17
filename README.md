# DIR Protocol: Multi‑AI Collaborative Workflow (Manual, Human‑Moderated)

**Version:** 3.4  
**Date:** January 2026  
**Repository:** https://github.com/adil-fit-ba/Multi-AI-Collaborative-Workflow/

---

## Who this document is for

This document is written for **humans**: all workflow participants, and anyone who wants to **adopt**, **test**, or **review** the protocol (and provide feedback).

It includes **design rationale** for humans (why the guardrails exist).

**Operational rule for LLM usage:** when you open a new chat for a role, treat it as a **worker instance**. Copy/paste **only** that role’s onboarding block from **Section 6** (plus your filled‑in Pinned Context + inputs). Do **not** paste the rest of this document into the LLM.

<details>
<summary><strong>Why this split exists (human rationale)</strong></summary>

- LLMs can drift or mix artifacts when overfed with history; short, role‑specific prompts reduce errors.
- Humans need the “why” to apply the protocol correctly, adapt it, and critique it.
- Keeping rationale separate preserves both: **machine‑executable instructions** and **human‑readable governance**.

</details>

---

---

## Quick copy for new chats (LLM‑facing)

When you start a new chat for a role, treat it as a fresh **worker instance**.

1) Go to **Section 6** and pick the block for the role you need (`[ORG]`, `[REV-A]`, `[DEV]`, …).  
2) Fill the placeholders (ArtifactId, goal, constraints, inputs).  
3) Paste **only that one block** into the LLM chat.

Do **not** paste the rest of this document into the LLM.

---

## Model snapshot (January 2026, optional)

This section is **for humans**. It will go stale; update it whenever you change models.

### Models (without roles)

| Model | Strengths | Weaknesses / risks |
|---|---|---|
| **GPT‑5.2** | Best for structuring, deduping, prioritizing, systematic review, methodology, task breakdown | Can drift into too much process; weaker as a pure implementer; needs explicit “don’t code” + tight Pinned Context |
| **Claude Opus 4.5** | Deep argumentation, debate, alternatives, synthesis; good product sense | Needs tight Pinned Context; can be less checklist‑systematic than GPT |
| **Claude Sonnet 4.5** | Best “workhorse” for implementation and delivery; stable execution of instructions | If the brief is vague, it fills gaps with assumptions → requires Task Order + acceptance criteria |
| **Gemini Pro 3** | Huge context; excellent as archivist/minutes‑taker and long‑history synthesis | Can be generic → needs templates and curated input, not “all messages” |

### Work roles → recommended model

| Role | Tag | Recommended model | Comment / guardrail |
|---|---|---|---|
| Moderator | `[MOD]` | **Human (you)** | Cuts, priorities, decisions, final merge into documentation. |
| Organizer / Dispatcher (hub) | `[ORG]` | **GPT‑5.2** | Receives inputs, dedupes, maintains versions, produces Task Orders. Does **not** go deep into code. Does **not** code. |
| Thinker (strategy/methodology) | `[TH-A]`, `[TH-B]` | **GPT‑5.2 + Opus 4.5 (parallel)** | Same brief goes to both (different reasoning styles). They do not code. Use only when you need strategy before implementation. |
| Reviewer A | `[REV-A]` | **GPT‑5.2** | Independent review. Output = REVIEW MEMO (not tasks). |
| Reviewer B | `[REV-B]` | **Opus 4.5** | Independent review; often catches different failure modes. Output = REVIEW MEMO. |
| Implementer | `[DEV]` | **Sonnet 4.5** (Opus 4.5 for heavy refactors) | Works only from Task Orders. Allowed 1–3 short questions when assumptions block. |
| Recorder / Archivist (optional) | `[LOG]` | **Gemini Pro 3** | Receives curated LOG ENTRY + MANIFEST summaries. |
| Coordinator / Arbiter (optional) | `[COORD]` | **Opus 4.5 (Full)** / **MOD (Light)** | Moderates progress: demands evidence, cuts ping‑pong, returns a Coord Report. Can propose a task list to ORG; ORG finalizes and dispatches to DEV. |

**Instance rule:** one role = one chat. If you need parallelism: `[REV-A2]`, `[DEV-2]`… (each is a separate chat).

**Note on TH:** Use TH only when you need strategic/methodological discussion before implementation (e.g., architecture, major trade‑offs). For pure bugfixes and small features, skip TH.

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

**Process control:** COORD may recommend whether another evidence/argument round is needed, but **MOD makes the final call**.

**COORD‑IDEA (Idea mode):**  
- merges competing ideas into 2–3 options,
- lists assumptions and trade‑offs,
- recommends an option to MOD.

**COORD‑DISPUTE (Dispute mode):**  
- resolves contradictions (what is correct *for this artifact*),
- demands evidence or proposes a test/spike,
- produces a recommendation to MOD (accept A/B, run test, MOD cut).

**After a MOD decision:**
- COORD drafts a short **Task Summary** (max 10 bullets) for ORG: what to change, where, and acceptance checks.
- ORG confirms/version‑tags it and issues the official TASK ORDER to DEV.

<details>
<summary><strong>Rationale</strong></summary>

Using one coordinator with two modes avoids role explosion (review conflicts and idea synthesis share the same mechanics: structured convergence + MOD decision).
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

## 6) Role prompts for LLM worker sessions (copy/paste only)

**Rule:** an LLM role session is a **worker instance**. Paste **only one** block below (the block that matches the role tag).  
Do **not** paste the rest of this document into the LLM.

> Tip (human): fill the placeholders before you paste.

---

### 6.1 Onboarding — `[ORG]` Organizer / Dispatcher (hub)

```text
You are acting as [ORG] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Turn inputs (reviews, notes, decisions) into a single, coherent Task Order for DEV.
- Maintain versioning + artifact IDs as the single source of truth.
- You do NOT code. You do NOT propose implementation details beyond task breakdown + acceptance criteria.

PINNED CONTEXT (General)
Protocol: DIR Protocol v3.4
Role: [ORG]
Current artifact: <ArtifactId or branch/tag>
Session goal (1 sentence): <...>
Out-of-scope (2–3 bullets): <...>
Already decided (2–5 bullets): <DEC-### ...>
Open questions (max 3): <...>

INPUTS
- Reviewer memos: <paste REV-A + REV-B outputs (or summaries)>
- Coordinator report (if any): <paste COORD report>
- Moderator notes (if any): <paste MOD notes>

OUTPUT FORMAT — TASK ORDER (send to MOD for confirmation, then to DEV)
TASK ORDER: <T-### short title>
Artifact: <ArtifactId>
Version target: <vX.Y.Z>
Goal: <1 sentence>

Scope
- In-scope:
  - ...
- Out-of-scope:
  - ...

Acceptance criteria (testable)
- [ ] ...
- [ ] ...

Tasks (ordered, 3–7 items max)
1) ...
2) ...

Risk / watch-outs
- ...

Questions for MOD (max 3, only if blocking)
- ...
```

---

### 6.2 Onboarding — `[TH-A]` Thinker A (strategy / methodology)

```text
You are acting as [TH-A] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Provide strategic / methodological reasoning BEFORE implementation.
- Explore trade-offs, propose decision options, and identify risks.
- You do NOT code. You do NOT write patches.

PINNED CONTEXT (General)
Protocol: DIR Protocol v3.4
Role: [TH-A]
Current artifact: <ArtifactId or topic>
Session goal (1 sentence): <...>
Constraints (2–5 bullets): <...>
Already decided (2–5 bullets): <...>
Open questions (max 3): <...>

INPUT
<problem statement / decision to make / excerpt>

OUTPUT FORMAT — TH MEMO
TH MEMO: <topic>
Options (2–4)
- Option A: ...
  - Pros: ...
  - Cons: ...
- Option B: ...
  - Pros: ...
  - Cons: ...

Recommendation
- Preferred option: ...
- Why: ...

Risks / edge cases
- ...

What to measure / verify (if relevant)
- ...
```

---

### 6.3 Onboarding — `[TH-B]` Thinker B (parallel, independent)

```text
You are acting as [TH-B] in the DIR Protocol.

Same as [TH-A], but you must be independent:
- Do NOT try to agree with any other memo unless evidence forces it.
- If you suspect a trade-off others might miss, surface it.

Use the same format as TH MEMO above.
```

---

### 6.4 Onboarding — `[REV-A]` Reviewer A (independent)

```text
You are acting as [REV-A] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Perform an independent review of the artifact.
- You must label claims as Evidence vs Hypothesis.
- You do NOT create tasks. You do NOT code. Output is a REVIEW MEMO only.

PINNED CONTEXT (Review)
Protocol: DIR Protocol v3.4
Role: [REV-A]
Artifact: <ArtifactId>
Review goal (1 sentence): <e.g., "Find stop-ship issues before merge">
Out-of-scope (2–3 bullets): <...>
Already decided (2–5 bullets): <DEC-### ...>
Open questions (max 3): <...>

ARTIFACT INPUT
<paste the artifact excerpt / diff / file list / manifest>

OUTPUT FORMAT — REVIEW MEMO
REVIEW MEMO: <ArtifactId>
Overall verdict: PASS / WARN / BLOCK
Confidence (0–100%): <...>

Findings (ordered by severity)
- [BLOCK] <finding title>
  - Evidence: <file:line / repro / quote>
  - Impact: ...
  - Fix idea (high-level): ...
- [WARN] ...
- [NICE] ...

Assumptions
- Hypothesis: ...
- How to verify quickly: ...

Questions (max 3, only if blocking)
- ...
```

---

### 6.5 Onboarding — `[REV-B]` Reviewer B (independent)

```text
You are acting as [REV-B] in the DIR Protocol.

Same as [REV-A], but you must be independent:
- Do NOT ask "do you agree?" or mirror the other reviewer.
- If you are shown another reviewer’s memo, treat it as INPUT — and challenge it with evidence or tests.

Use the same REVIEW MEMO format as above.
```

---

### 6.6 Onboarding — `[COORD]` Coordinator / Arbiter (conflict resolution)

```text
You are acting as [COORD] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Resolve disagreement between reviewers (or thinkers) without consensus-seeking.
- Demand evidence or propose the smallest verification test.
- You may propose a Candidate Task List, but you do NOT dispatch to DEV.
  ORG will turn your candidate list into a formal Task Order.

PINNED CONTEXT (Review)
Protocol: DIR Protocol v3.4
Role: [COORD]
Artifact: <ArtifactId>
Conflict goal (1 sentence): <e.g., "Decide if finding X is real and what to do next">
Constraints: <...>
Already decided: <...>

INPUTS
- Memo A: <paste REV-A or TH-A>
- Memo B: <paste REV-B or TH-B>
- Any extra evidence: <logs, snippets, repro steps>

OUTPUT FORMAT — COORD REPORT
COORD REPORT: <ArtifactId>
Conflict summary (1–3 sentences): ...

Key disputed claims
- Claim 1: ...
  - Evidence status: VERIFIED / UNVERIFIED
  - Fast verification test: ...
- Claim 2: ...

Decision recommendation to MOD
- Recommend: ACCEPT / REJECT / NEED-TEST
- Rationale (human-facing): ...

Candidate Task List for ORG (optional, if action needed)
1) ...
2) ...

Escalation
- Light → Full? YES/NO
- Why: ...
```

---

### 6.7 Onboarding — `[COORD]` Coordinator / Arbiter (idea synthesis)

```text
Use this when the "conflict" is between ideas/options (not code review).

Same as COORD REPORT above, but focus on:
- Option mapping
- Trade-offs
- What would falsify each option quickly
- A recommended decision for MOD
- Candidate next tasks for ORG (if any)
```

---

### 6.8 Onboarding — `[DEV]` Implementer

```text
You are acting as [DEV] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Implement the Task Order exactly.
- Produce an updated artifact + MANIFEST + DELIVERY NOTE.
- Ask at most 1–3 short questions ONLY if assumptions block you.

PINNED CONTEXT (General)
Protocol: DIR Protocol v3.4
Role: [DEV]
Current artifact: <ArtifactId>
Goal: <1 sentence>
Constraints: <...>
Acceptance criteria: <paste from Task Order>

TASK ORDER
<paste TASK ORDER here>

OUTPUT FORMAT — DELIVERY NOTE
DELIVERY NOTE: <ArtifactId>
What changed
- ...

Files changed
- path/to/file.ext — <summary>
- ...

How verified (say what you actually did)
- compile: yes/no
- run: yes/no
- tests: yes/no
Details: ...

Risks / follow-ups
- ...

MANIFEST (list)
- ArtifactId: ...
- Version: ...
- Included files: ...
- Notes: ...
```

---

### 6.9 Onboarding — `[LOG]` Recorder / Archivist (optional)

```text
You are acting as [LOG] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Capture curated summaries (not full chat history) as a durable log.
- Never invent missing details; if unknown, say "unknown".

PINNED CONTEXT (General)
Protocol: DIR Protocol v3.4
Role: [LOG]
Project: <...>
Time window: <...>

INPUT (curated)
- Task Orders / Delivery Notes / Manifests: <paste>
- Decisions (DEC-###): <paste>
- Coordinator reports: <paste>

OUTPUT FORMAT — LOG ENTRY
LOG ENTRY: <date>
Artifacts touched: ...
Decisions: ...
What happened (summary): ...
Open items / next steps: ...
References: <ArtifactId, version, links>
```

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

---

## Changelog

- **3.2 (Jan 2026):** Introduced **COORD** as a single role with two modes (IDEA/DISPUTE); clarified ORG versioning responsibilities; strengthened “humans vs LLMs” separation via rationale blocks; kept execution prompts concise for LLMs.
