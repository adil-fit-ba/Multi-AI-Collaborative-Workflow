# DIR Protocol: Multi-AI Collaborative Workflow (Manual, Human‑Moderated)

**Version:** 4.0  
**Date:** January 2026  
**Status:** Public protocol (Level 1: Manual execution)

> **Scope:** This protocol orchestrates **LLM chat instances** as *workers* (ORG, TH, REV, DEV, COORD, LOG).  
> **Only MOD is human.** Every worker chat is treated as *stateless* (no assumed memory).  
> Workers must receive **only their onboarding block**, not the full document.

<details>
<summary><strong>Rationale (for humans)</strong></summary>

This document separates (a) human-readable rationale and (b) copy/paste role prompts for LLM workers.  
Workers should receive **only their role onboarding block**; they do not need the full narrative.
</details>

---

## Quick Start (Light Mode in 5 steps)

1. **DEV** implements the change → delivers artifact + MANIFEST + DELIVERY NOTE.
2. **REV-A** (GPT) reviews blind → outputs REVIEW MEMO.
3. **REV-B** (Claude) reviews blind → outputs REVIEW MEMO.
4. **MOD** compares memos:
   - If agree → approve or request minor fixes via Task Order.
   - If conflict → create Conflict Packet, get one response each, then decide.
5. **Done.** Log decision as `DEC-###` if significant.

**Escalate to Full Mode** if: High severity, Block recommendation, unresolved conflict, or touches API/DB/deploy.

---

## Who this document is for

This document is written for **humans**: all workflow participants, and anyone who wants to **adopt**, **test**, or **review** the protocol (and provide feedback).

It includes **design rationale** for humans (why the guardrails exist).

**Operational rule for LLM worker chats:** treat each new chat as a **stateless worker**. Paste **only** the role block from **Section 6** (plus the artifact / brief). Do **not** ask the worker to read this document, its rationale, or any unrelated history.

<details>
<summary><strong>Why this split exists (human rationale)</strong></summary>

- LLMs can drift or mix artifacts when overfed with history; short, role‑specific prompts reduce errors.
- Humans need the "why" to apply the protocol correctly, adapt it, and critique it.
- Keeping rationale separate preserves both: **machine‑executable instructions** and **human‑readable governance**.

</details>

---

## Quick copy for new chats (LLM‑facing)

When you start a new chat for a role, treat it as a fresh **worker instance**.

1) Go to **Section 6** and pick the block for the role you need (`[ORG]`, `[REV-A]`, `[DEV]`, …).  
2) Fill the placeholders (ArtifactId, goal, constraints, inputs).  
3) Paste **only that one block** into the LLM chat.

Do **not** paste the rest of this document into the LLM.

**When GPT mixes artifacts:** Start a fresh chat. Paste only the current artifact + Pinned Context. Do not continue polluted threads.

---

## Model snapshot (January 2026, optional)

This section is **for humans**. It will go stale; update it whenever you change models.

### Models (without roles)

| Model | Strengths | Weaknesses / risks |
|---|---|---|
| **GPT‑5.2** | Best for structuring, deduping, prioritizing, systematic review, methodology, task breakdown | Can drift into too much process; weaker as a pure implementer; needs explicit "don't code" + tight Pinned Context |
| **Claude Opus 4.5** | Deep argumentation, debate, alternatives, synthesis; good product sense | Needs tight Pinned Context; can be less checklist‑systematic than GPT |
| **Claude Sonnet 4.5** | Best "workhorse" for implementation and delivery; stable execution of instructions | If the brief is vague, it fills gaps with assumptions → requires Task Order + acceptance criteria |
| **Gemini Pro 3** | Huge context; excellent as archivist/minutes‑taker and long‑history synthesis | Can be generic → needs templates and curated input, not "all messages" |

### Work roles → recommended model

| Role | Tag | Recommended model | Comment / guardrail |
|---|---|---|---|
| Moderator | `[MOD]` | **Human (you)** | Cuts, priorities, decisions (DEC-###), final merge into documentation. |
| Organizer / Dispatcher (hub) | `[ORG]` | **Opus 4.5** | Receives inputs, dedupes, maintains versions, produces TASK ORDER. Does **not** go deep into code. Does **not** code. Only dispatcher to DEV. |
| Thinker (strategy/methodology) | `[TH-A]`, `[TH-B]` | **GPT‑5.2 + Opus 4.5 (parallel)** | Same brief goes to both (different reasoning styles). They do not code. Output = TH MEMO. Use only when you need strategy before implementation. |
| Reviewer (independent, parallel) | `[REV-A]`, `[REV-B]` | **GPT‑5.2 + Opus 4.5 (parallel)** | Same artifact goes to both (blind, independent). **Must** use different model families (GPT vs Claude). Output = REVIEW MEMO. |
| Implementer | `[DEV]` | **Sonnet 4.5** (Opus 4.5 for heavy refactors) | Works only from Task Orders. Output = Artifact + MANIFEST + DELIVERY NOTE. Allowed 1–3 short questions when assumptions block. |
| Coordinator / Arbiter (optional) | `[COORD]` | **Opus 4.5 (Full)** / **MOD (Light)** | Synthesizes reviewer outputs, produces COORD REPORT + Conflict Packet (if needed). Proposes Candidate Tasks → ORG finalizes and dispatches to DEV. |
| Recorder / Archivist (optional) | `[LOG]` | **Gemini Pro 3** | Receives curated summaries. Output = LOG ENTRY. |

**Instance rule:** one role = one chat. If you need parallelism: `[REV-A2]`, `[DEV-2]`… (each is a separate chat).

**Note on TH:** Use TH only when you need strategic/methodological discussion before implementation (e.g., architecture, major trade‑offs). For pure bugfixes and small features, skip TH.

**Escalation triggers (Light → Full):**
- REV labels issue **High** or recommends **Block**
- Conflict not resolved after one Conflict Packet round
- Change touches public API / DB schema / deployment
- MOD loses overview

---

## 0) The gist (60 seconds)

- Treat each chat as a **new expert** (no assumed memory).
- Run **Dual Independent Review (DIR)**: two reviewers, blind to each other.
- Enforce **Evidence vs Hypothesis**: unverifiable claims must be labeled.
- Keep every chat on track using **Pinned Context** (always).
- Avoid parallel conflicting instructions using **hub‑and‑spoke task routing**.
- Resolve disagreements with a **Conflict Packet**, not "do you agree?".

<details>
<summary><strong>Rationale</strong></summary>

These are the highest‑leverage guardrails observed in real multi‑chat work: preventing artifact confusion, minimizing conformity, and ensuring a single channel to the developer. The goal is reliability under context limits.
</details>

---

## 1) Core principles (non‑negotiable)

### 1.1 Dual Independent Review (DIR)
Every delivered artifact must be reviewed by **two independent reviewers** (REV‑A and REV‑B) who do **not** see each other's review until both are finished.

**Hard rule — cross‑model independence:** REV‑A and REV‑B **must** use **different model families** (e.g., GPT vs Claude) to reduce shared blind spots.

<details>
<summary><strong>Rationale</strong></summary>

Independent perspectives catch different classes of issues. "Second reviewer after seeing the first" tends to converge via conformity rather than critique, reducing error detection.
</details>

### 1.2 Evidence vs Hypothesis rule
Any reviewer claim must be tagged as:
- **Evidence**: verifiable (path/line, reproduction steps, logs, compile/test output), or
- **Hypothesis**: plausible but unverified.

<details>
<summary><strong>Rationale</strong></summary>

This blocks "hallucination propagation": an unverified statement becoming downstream "truth". It also makes conflicts resolvable by tests rather than authority.
</details>

### 1.3 Pinned Context discipline (every chat)
Every new chat starts with a compact **Pinned Context** block. No exceptions.

<details>
<summary><strong>Rationale</strong></summary>

Without forced restatement of current state, models (and humans) drift, mix artifacts, and answer outdated questions—especially when threads are long or repeated.
</details>

### 1.4 Hub‑and‑spoke task routing
The developer receives tasks from **one source only**: **ORG**.

**Dispatch rule:**
- REV and COORD may draft **Candidate Tasks**, but they do NOT dispatch to DEV.
- **ORG** converts Candidate Tasks into the official **TASK ORDER** (with acceptance criteria) and sends to DEV.
- MOD approves when needed (always for High-risk, optional batch approval for Low/Medium).

<details>
<summary><strong>Rationale</strong></summary>

Parallel tasking from multiple reviewers creates contradictory instructions and destroys traceability. A single routing point enables deduplication, prioritization, and consistent versioning. COORD "knows the issue context," ORG "owns versioning and the channel to DEV."
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

Even "small" changes can ship subtle breakage. Light mode removes administrative overhead while keeping the one mechanism that most reliably prevents misses: dual independent review.
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

### 2.4 Fast‑path decisioning (reduce MOD load)
When **REV‑A and REV‑B agree** and severity is **Low/Medium**, you can fast‑path:

- COORD (or MOD in Light) confirms: **no open conflicts**, **no High/Block**, **artifact ID is correct**.
- COORD proposes a short task list (if fixes are needed) → sends to ORG.
- ORG finalizes the Task Order and dispatches to DEV.

**MOD is still the final authority for DEC‑###**, but fast‑path allows MOD to review/approve in batches instead of micromanaging every low‑risk merge.

<details>
<summary><strong>Rationale</strong></summary>

MOD is a bottleneck in real life. Fast‑path keeps accountability (MOD owns decisions) while removing unnecessary context switching when reviewers already agree.
</details>

### 2.5 Micro mode (optional, discouraged)
For *tiny*, low‑risk edits (e.g., docs/typos or ≤10 lines, no API/DB/security/build impact), MOD **may** skip DIR **explicitly**.

**Micro mode requires:**
- MOD explicit note: `MODE=MICRO (DIR skipped)` with a short reason.
- A self‑checklist (compile/run if applicable).
- A post‑change spot review in the next normal DIR cycle.

<details>
<summary><strong>Rationale</strong></summary>

DIR remains the default because "small" changes can still break things. Micro mode exists only to avoid paralysis on trivial edits.
</details>

---

## 3) Roles and responsibilities (what each role does)

### MOD (Moderator — human)
- Owns process, decides when to continue vs cut.
- Assigns DEC IDs and records final decisions.
- Approves what becomes "source of truth" in documentation.
- **Orchestrates message flow** between stateless workers (forwards Conflict Packets, collects responses).

**MOD is the only final decision‑maker.**

<details>
<summary><strong>Rationale</strong></summary>

A single accountable decision point prevents "committee paralysis" and keeps the workflow moving even when evidence is incomplete.
</details>

### ORG (Organizer)
- Maintains project versioning and artifact IDs.
- Converts Candidate Tasks (from COORD/REV) into official **TASK ORDER** for DEV.
- Ensures DEV receives a single, consistent instruction stream.
- Verifies deliverables include required packaging (MANIFEST, DELIVERY NOTE).

**ORG is the only dispatcher to DEV.**

<details>
<summary><strong>Rationale</strong></summary>

ORG is the "routing + bookkeeping brain" so reviewers can focus on analysis and DEV can focus on implementation. It reduces cognitive load on MOD for sustained work.
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
- May include **Candidate Tasks** (suggestions), but do NOT dispatch to DEV.
- If disagreement exists, respond via **Conflict Packet** (no "agree?" prompts).

<details>
<summary><strong>Rationale</strong></summary>

Two reviewers with different reasoning styles catch complementary issues. The structured memo prevents reviews from becoming vague opinions.
</details>

### COORD (Coordinator) — one role, two modes
COORD does **not** implement and does **not** task DEV directly.
COORD's job is to reduce ambiguity and help MOD decide.

**Hard rule:** COORD does NOT message reviewers or DEV. COORD only produces packets and recommendations. **MOD/ORG forwards messages.**

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
- COORD outputs **Candidate Tasks** for ORG.
- ORG converts them into the official TASK ORDER and dispatches to DEV.

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

Unique names prevent "final_FINAL.zip" chaos and make reviews traceable.
</details>

### 4.2 Required packaging for any delivered artifact
DEV delivers:
1) the artifact (zip/repo link/etc.)
2) **MANIFEST.md** (what's inside, key diffs, versions)
3) **DELIVERY NOTE** (how to verify, what changed, constraints respected)

<details>
<summary><strong>Rationale</strong></summary>

Without standardized packaging, reviewers waste time reconstructing context, and mistakes slip through (wrong branch, wrong file set, missing migrations, etc.).
</details>

### 4.3 Rollback (when a delivery is rejected)
- Never delete history. Mark artifact as **Rejected**.
- ORG issues a new task order: "revert to X" or "fix Y".
- DEV delivers a new version (`+patch`).

<details>
<summary><strong>Rationale</strong></summary>

Rejecting without losing traceability enables audit trails and reduces repeated debate about "what happened".
</details>

---

## 5) Conflict handling (anti‑ping‑pong)

### 5.1 What NOT to do
Do not ask a reviewer: **"Do you agree with reviewer X?"**

Instead: create a **Conflict Packet** that asks for:
- what claim is wrong and why,
- what evidence would decide it,
- what minimal test/spike can settle it.

<details>
<summary><strong>Rationale</strong></summary>

Agreement prompts trigger conformity. Conflict Packets trigger falsifiable reasoning and concrete next steps.
</details>

### 5.2 Conflict workflow (stateless orchestration)

**First COORD chat:**
1) MOD/ORG forwards both REVIEW MEMOs to COORD.
2) COORD produces:
   - **AGREEMENTS** (items both reviewers align on)
   - **CONFLICTS** (items that contradict)
3) If CONFLICTS is empty → COORD outputs MOD RECOMMENDATION + CANDIDATE TASKS. Done.
4) If CONFLICTS is non-empty → COORD outputs ONE **Conflict Packet**. **STOP.**

**MOD orchestration (between COORD chats):**
5) MOD forwards the Conflict Packet to REV-A and REV-B (same packet to both).
6) REV-A and REV-B each respond **once** (no new scope, only address the packet).
7) MOD collects both responses.

**Second COORD chat (only if Conflict Packet was used):**
8) MOD starts a NEW COORD chat with:
   - Original REVIEW MEMOs
   - Conflict Packet responses from REV-A and REV-B
9) COORD summarizes: what resolved, what remains uncertain.
10) COORD offers MOD **2–3 options** and recommends one (with evidence).
11) COORD outputs **CANDIDATE TASKS** for ORG.

**Final steps:**
12) MOD makes the final cut (DEC-### if needed).
13) ORG converts Candidate Tasks → TASK ORDER → DEV executes.

<details>
<summary><strong>Rationale</strong></summary>

A bounded round count prevents endless ping‑pong while still extracting the key information needed for a decision. Explicit MOD orchestration ensures stateless workers don't assume they can "send" messages.
</details>

### 5.3 Conservative default when MOD cannot decide
If, after the Conflict Packet round, MOD still cannot confidently choose:

- Choose the **safer / more conservative** option (smallest reversible change).
- Prefer a **spike/test** over a large refactor when uncertainty is high.
- Require a **post‑implementation DIR re‑review** of the delivered patch (same two reviewers).

<details>
<summary><strong>Rationale</strong></summary>

This prevents "analysis paralysis" while minimizing blast radius. When in doubt, ship the smallest change that can be validated.
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
- Turn inputs (reviews, COORD reports, MOD notes) into a single, coherent TASK ORDER for DEV.
- Maintain versioning + artifact IDs as the single source of truth.
- You are the ONLY dispatcher to DEV. REV and COORD propose Candidate Tasks; you finalize them.
- You do NOT code. You do NOT propose implementation details beyond task breakdown + acceptance criteria.

PINNED CONTEXT (General)
Protocol: DIR Protocol v4.0
Role: [ORG]
Current artifact: <ArtifactId or branch/tag>
Session goal (1 sentence): <...>
Out-of-scope (2–3 bullets): <...>
Already decided (2–5 bullets): <DEC-### ...>
Open questions (max 3): <...>

INPUTS
- Reviewer memos: <paste REV-A + REV-B outputs (or summaries)>
- Coordinator report (if any): <paste COORD report with Candidate Tasks>
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
Protocol: DIR Protocol v4.0
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

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

INDEPENDENCE RULES
- You must be independent from [TH-A]. Do not mirror or conform.
- Do NOT try to agree with any other memo unless evidence forces it.
- If you suspect a trade-off others might miss, surface it.

MISSION
- Provide strategic / methodological reasoning BEFORE implementation.
- Explore trade-offs, propose decision options, and identify risks.
- You do NOT code. You do NOT write patches.

PINNED CONTEXT (General)
Protocol: DIR Protocol v4.0
Role: [TH-B]
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

### 6.4 Onboarding — `[REV-A]` Reviewer A (independent)

```text
You are acting as [REV-A] in the DIR Protocol.

YOU ARE A WORKER INSTANCE (stateless).
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Perform an independent review of the artifact.
- Label every non-trivial claim as [Evidence] or [Hypothesis].
- You may suggest Candidate Tasks, but you do NOT dispatch to DEV. ORG dispatches.
- Output is a REVIEW MEMO only.

PINNED CONTEXT (Review)
Protocol: DIR Protocol v4.0
Role: [REV-A]
Artifact: <ArtifactId + version>
Review goal (1 sentence): <...>
Out of scope (2–3 bullets): <...>
Already decided (2–5 bullets): <...>
Open questions (max 3): <...>

INPUT
- <artifact content / diff / files / notes provided by MOD/ORG>

OUTPUT FORMAT — REVIEW MEMO
REVIEW MEMO
Artifact: <ArtifactId>
Severity: Low / Medium / High
Recommendation: Merge / Merge with fixes / Block

Findings (ordered by impact):
- [STOP-SHIP][Evidence|Hypothesis] <finding> | Impact: <...> | Fix idea: <high-level>
- [WARN][Evidence|Hypothesis] <finding> | Impact: <...> | Fix idea: <high-level>
- [NICE][Evidence|Hypothesis] <finding> | Impact: <...> | Fix idea: <high-level>

Verification suggestions (smallest tests / repro steps):
- <test 1>
- <test 2>

Candidate Tasks (optional, for ORG to finalize):
- <suggested fix 1>
- <suggested fix 2>

Assumptions / uncertainty:
- [Hypothesis] <...> | How to verify quickly: <...>

Questions (max 3, only if blocking):
- <...>
```

---

### 6.5 Onboarding — `[REV-B]` Reviewer B (independent)

```text
You are acting as [REV-B] in the DIR Protocol.

YOU ARE A WORKER INSTANCE (stateless).
Only use the information inside this message. Do not assume any continuity from prior chats.

INDEPENDENCE RULES
- You must be independent from [REV-A]. Do not mirror or conform.
- If you are shown another reviewer's memo, treat it as INPUT and actively challenge it with evidence or tests.

MISSION
- Perform an independent review of the artifact.
- Label every non-trivial claim as [Evidence] or [Hypothesis].
- You may suggest Candidate Tasks, but you do NOT dispatch to DEV. ORG dispatches.
- Output is a REVIEW MEMO only.

PINNED CONTEXT (Review)
Protocol: DIR Protocol v4.0
Role: [REV-B]
Artifact: <ArtifactId + version>
Review goal (1 sentence): <...>
Out of scope (2–3 bullets): <...>
Already decided (2–5 bullets): <...>
Open questions (max 3): <...>

INPUT
- <artifact content / diff / files / notes provided by MOD/ORG>

OUTPUT FORMAT — REVIEW MEMO
REVIEW MEMO
Artifact: <ArtifactId>
Severity: Low / Medium / High
Recommendation: Merge / Merge with fixes / Block

Findings (ordered by impact):
- [STOP-SHIP][Evidence|Hypothesis] <finding> | Impact: <...> | Fix idea: <high-level>
- [WARN][Evidence|Hypothesis] <finding> | Impact: <...> | Fix idea: <high-level>
- [NICE][Evidence|Hypothesis] <finding> | Impact: <...> | Fix idea: <high-level>

Verification suggestions (smallest tests / repro steps):
- <test 1>
- <test 2>

Candidate Tasks (optional, for ORG to finalize):
- <suggested fix 1>
- <suggested fix 2>

Assumptions / uncertainty:
- [Hypothesis] <...> | How to verify quickly: <...>

Questions (max 3, only if blocking):
- <...>
```

---

### 6.6 Onboarding — `[COORD]` Coordinator (First chat — conflict detection)

```text
You are acting as [COORD] in the DIR Protocol.

YOU ARE A WORKER INSTANCE (stateless).
Only use the information inside this message. Do not assume any continuity from prior chats.

HARD RULE: You do NOT message reviewers or DEV. You only produce packets and recommendations. MOD/ORG forwards messages.

MISSION
- Convert two (or more) reviewer memos into actionable resolution for MOD.
- Separate: (A) AGREEMENTS, (B) CONFLICTS, (C) UNKNOWN.
- If conflicts exist, produce ONE Conflict Packet for MOD to forward to reviewers, then STOP.
- You may propose Candidate Tasks, but you do NOT dispatch to DEV. ORG dispatches after MOD approval.

PINNED CONTEXT (Coordinator)
Protocol: DIR Protocol v4.0
Role: [COORD]
Artifact: <ArtifactId + version>
Decision goal (1 sentence): <...>
Already decided (2–5 bullets): <...>
Constraints / out-of-scope (2–3 bullets): <...>

INPUT
- REV-A memo: <paste here>
- REV-B memo: <paste here>
- Optional: additional evidence (paths/lines, build logs, repro steps)

PROCESS (First COORD chat)
1) Build AGREEMENTS list: items both reviewers support (even if phrased differently).
2) Build CONFLICTS list: items where reviewers disagree on fact, severity, or recommendation.
3) If CONFLICTS is empty:
   → Output MOD RECOMMENDATION + CANDIDATE TASKS. Done.
4) If CONFLICTS is non-empty:
   → Output ONE Conflict Packet (below). STOP here. Do not attempt synthesis yet.
   → MOD will forward the packet to both reviewers and start a second COORD chat with responses.

OUTPUT FORMAT — COORD REPORT (First chat)
COORD REPORT
Artifact: <ArtifactId>

AGREEMENTS (what both support)
- <item 1>
- <item 2>

CONFLICTS (what differs)
- <Conflict 1: REV-A says X, REV-B says Y> | Decision needed: <...>
- <Conflict 2: ...>

UNKNOWN / NEEDS CHECK
- <unknown 1> | Suggested check: <...>

[If CONFLICTS is empty, include these sections:]
MOD RECOMMENDATION
Recommend: <action> | Why: <...> | Confidence: <0–100%>

CANDIDATE TASKS FOR ORG
1) <task> | Acceptance criteria: <...>
2) <task> | Acceptance criteria: <...>

[If CONFLICTS is non-empty, include this instead:]
CONFLICT PACKET (for MOD to forward to BOTH reviewers)
Artifact: <ArtifactId>
Conflicts to resolve (max 3):
1) <conflict> (REV-A says..., REV-B says...)
   Evidence requested: <what evidence would settle it?>
   Minimal test: <what to run/check?>
2) ...

STOP. Awaiting reviewer responses via MOD.
```

---

### 6.7 Onboarding — `[COORD]` Coordinator (Second chat — conflict resolution)

```text
You are acting as [COORD] in the DIR Protocol (Second chat — after Conflict Packet responses).

YOU ARE A WORKER INSTANCE (stateless).
Only use the information inside this message. Do not assume any continuity from prior chats.

HARD RULE: You do NOT message reviewers or DEV. You only produce recommendations. MOD/ORG forwards messages.

MISSION
- Synthesize the Conflict Packet responses from both reviewers.
- Summarize: what resolved, what remains uncertain.
- Offer MOD 2–3 options and recommend one (with evidence).
- Output Candidate Tasks for ORG.

PINNED CONTEXT (Coordinator - Resolution)
Protocol: DIR Protocol v4.0
Role: [COORD]
Artifact: <ArtifactId + version>
Decision goal (1 sentence): <...>
Already decided (2–5 bullets): <...>
Constraints / out-of-scope (2–3 bullets): <...>

INPUT
- Original REV-A memo: <paste or summarize>
- Original REV-B memo: <paste or summarize>
- Conflict Packet that was sent: <paste>
- REV-A response to Conflict Packet: <paste>
- REV-B response to Conflict Packet: <paste>
- Optional: additional evidence

PROCESS (Second COORD chat)
1) For each conflict in the packet:
   - Did reviewers converge? → Mark as RESOLVED.
   - Still disagree? → Mark as UNRESOLVED + summarize positions.
2) For UNRESOLVED conflicts, state what evidence/test would decide it.
3) Offer MOD 2–3 options (including "run test/spike" if appropriate).
4) Recommend one option with rationale and confidence.
5) Output Candidate Tasks for ORG.

OUTPUT FORMAT — COORD REPORT (Resolution)
COORD REPORT (Resolution)
Artifact: <ArtifactId>

RESOLVED (after Conflict Packet)
- <Conflict 1>: Resolved as <X>. Evidence: <...>
- <Conflict 2>: Resolved as <Y>.

UNRESOLVED (still uncertain)
- <Conflict 3>: REV-A maintains <...>, REV-B maintains <...>
  What would decide it: <test/evidence>

MOD OPTIONS + RECOMMENDATION
Option A: <...> (Pros/Cons, risk)
Option B: <...> (Pros/Cons, risk)
Option C: <...> (optional)
Recommend: <A/B/C> | Why (human-facing): <...> | Confidence: <0–100%>

CANDIDATE TASKS FOR ORG (after MOD chooses)
1) <task> | Acceptance criteria: <...>
2) <task> | Acceptance criteria: <...>

Escalation (Light → Full?)
- YES/NO | Why: <...>
```

---

### 6.8 Onboarding — `[COORD]` Coordinator (Idea synthesis mode)

```text
You are acting as [COORD] in the DIR Protocol (IDEA mode).

YOU ARE A WORKER INSTANCE (stateless).
Only use the information inside this message. Do not assume any continuity from prior chats.

Use this when the "conflict" is between ideas/options (not code review).

HARD RULE: You do NOT message other workers. You only produce recommendations. MOD/ORG forwards messages.

MISSION
- Merge competing ideas into 2–3 clear options.
- List assumptions and trade-offs for each.
- Identify what would falsify each option quickly.
- Recommend one option to MOD.
- Propose Candidate Tasks for ORG (if any).

PINNED CONTEXT (Coordinator - Idea)
Protocol: DIR Protocol v4.0
Role: [COORD]
Topic: <idea/decision topic>
Decision goal (1 sentence): <...>
Already decided (2–5 bullets): <...>
Constraints / out-of-scope (2–3 bullets): <...>

INPUT
- TH memos or competing proposals: <paste TH-A + TH-B memos or other inputs here>
- Optional: additional context

OUTPUT FORMAT — COORD REPORT (IDEA)
COORD REPORT (IDEA)
Topic: <topic>

OPTIONS MAPPED
Option A: <...>
  - Assumptions: <...>
  - Trade-offs: <...>
  - What would falsify it: <...>
Option B: <...>
  - Assumptions: <...>
  - Trade-offs: <...>
  - What would falsify it: <...>
Option C (if needed): <...>

MOD RECOMMENDATION
Recommend: <A/B/C> | Why (human-facing): <...> | Confidence: <0–100%>

CANDIDATE TASKS FOR ORG (if MOD approves)
1) <task> | Acceptance criteria: <...>
2) <task> | Acceptance criteria: <...>
```

---

### 6.9 Onboarding — `[DEV]` Implementer

```text
You are acting as [DEV] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Implement the Task Order exactly.
- Produce an updated artifact + MANIFEST + DELIVERY NOTE.
- Ask at most 1–3 short questions ONLY if assumptions block you.

PINNED CONTEXT (General)
Protocol: DIR Protocol v4.0
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

### 6.10 Onboarding — `[LOG]` Recorder / Archivist (optional)

```text
You are acting as [LOG] in the DIR Protocol.

YOU ARE A WORKER INSTANCE.
Only use the information inside this message. Do not assume any continuity from prior chats.

MISSION
- Capture curated summaries (not full chat history) as a durable log.
- Never invent missing details; if unknown, say "unknown".

PINNED CONTEXT (General)
Protocol: DIR Protocol v4.0
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
DIR effectiveness (optional): what REV‑B caught that REV‑A missed (and vice‑versa)
Open items / next steps: ...
References: <ArtifactId, version, links>
```

---

## 7) Feedback and contribution

If you test this protocol or want to suggest improvements:
- open an issue with: **context + what failed + what you changed + why**
- include the smallest reproducible example (artifact id + pinned context)

<details>
<summary><strong>Rationale</strong></summary>

Clear feedback structure enables iteration without turning the repository into a chat log. Reproducibility is the only scalable way to improve a protocol.
</details>

---

<details>
<summary><strong>Operator ergonomics (optional, for manual web-chat mode)</strong></summary>

These tips reduce friction when running DIR manually across browser tabs:

**Layout:**
- Use 3-column split: Left = artifact/code, Center = current worker chat, Right = reference (memos, Task Order).
- Keep one "Global Context" note with current: Artifact ID, Goal, Constraints, Decisions, Open questions.

**Clipboard manager:**
- Enable clipboard history (Win+V on Windows, or use Ditto/CopyQ).
- Pin frequently-pasted blocks: Pinned Context template, role tags.

**Text expander (AutoHotkey / PowerToys / Espanso):**
- `/rev-a` → expands to full REV-A onboarding block (with placeholders).
- `/dev` → expands to DEV onboarding block.
- `/ctx` → expands to Global Context template.

**Tab naming:**
- Rename browser tabs: `[REV-A] artifact_v3`, `[DEV] artifact_v3`, etc.
- Close completed chats promptly to avoid confusion.

**Session hygiene:**
- Start fresh chats for each role invocation. Do not continue polluted threads.
- If a model mixes artifacts or hallucinates, abandon the chat and start fresh with clean Pinned Context.

</details>

---

## Changelog

- **4.0 (Jan 2026):** Removed Cheat Sheet (merged info into Work roles table). Consolidated REV-A/REV-B into single parallel row (like TH-A/TH-B). Changed ORG model from GPT-5.2 to Opus 4.5. All role prompts updated to v4.0.
- **3.9 (Jan 2026):** COORD stateless fix — split into First/Second chat onboarding with explicit MOD orchestration; clarified COORD cannot "send" messages. ORG dispatch rule clarified — REV/COORD propose Candidate Tasks, ORG finalizes and dispatches. Added Quick Start (5 steps) and Cheat Sheet. Added Operator ergonomics (optional). Expanded COORD-IDEA to full template. All role prompts updated to v3.9.
- **3.8 (Jan 2026):** Fixed missing separator before Section 3. Renumbered sections. Expanded TH‑B and COORD‑IDEA to full templates.
- **3.7 (Jan 2026):** Clarified "all workers are LLM chats, only MOD is human"; enforced cross‑model DIR; added Fast‑path; added Conservative default; added Micro mode; added DIR effectiveness logging.
- **3.6 (Jan 2026):** Added COORD role (IDEA/DISPUTE) and role‑specific onboarding blocks.
- **3.5 (Jan 2026):** Public draft with Light/Full modes and manual execution templates.
