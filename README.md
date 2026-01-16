# Multi-AI Collaborative Workflow

A system for orchestrating multiple AI models in complex intellectual tasks.

---

## Philosophy

Each AI model has its strengths. Instead of relying on a single model, we use them as a **team of specialists** where each contributes what they do best.

**The user is the moderator of the process:**
- Decides when discussion moves forward, when to cut, and which messages/quotes enter the "Source of Truth"
- Copy/paste is curated — the moderator can shorten, reformulate, or merge messages before entering them into the document
- AI suggests structure and guardrails, but the moderator decides tempo, cuts, and final content

*This workflow is an assistive framework, not a rigid procedure.*

---

## Model Roles

| Model | Instance | Primary Role | Strengths | Weaknesses |
|-------|----------|--------------|-----------|------------|
| **Claude Opus ET** | Chat 1 | Strategist, debater | Deep analysis, discussion with GPT | Context limit |
| **Claude Opus/Sonnet** | Chat 2 | Implementer | Coding, writing, execution | Don't mix with discussion |
| **Claude Opus** | Chat 3 (optional) | Synthesis | Summarizing, conclusions | Alternative to Gemini |
| **GPT ET** | - | Strategist, critic | Deep analysis, review, methodology | Weaker at coding |
| **Gemini Pro** | - | Archivist, synthesis | Huge context (1M tokens) | More generic responses |

### ⚠️ Important Rule

**Claude instances DO NOT mix roles within the same chat:**
- Chat for discussion → ONLY discussion, no coding
- Chat for coding → ONLY coding, no strategic debates
- Chat for synthesis → ONLY summarizing and conclusions

### 📝 Note on Synthesis

Synthesis is an **optional phase**. It can be done by:
- Gemini Pro (if you need huge context)
- Claude Opus in a separate chat (if you don't use Gemini)
- Manually (you summarize into this document)

**Trigger for synthesis (recommendation):**
- Discussion exceeds ~10+ messages
- There are 2+ disagreements between models
- Switching to a new topic
- Returning after a break (day+)

---

## Task Types

### Software Development
- Architectural decisions
- Code review and refactoring
- API and database design
- Debugging complex problems

### Research
- Literature review
- Source synthesis
- Hypothesis development
- Methodological decisions

### Academic Work
- Dissertation/paper structure
- Argumentation
- Critical analysis
- Writing and revision

### Strategic Decisions
- Option evaluation
- Risk analysis
- Trade-off discussions
- Long-term planning

---

## Workflow

```
┌─────────────────────────────────────────────────────────┐
│          SYNTHESIS (Optional)                           │
│     Gemini Pro OR Claude Opus (separate chat)           │
│         Holds full context, synthesizes, remembers     │
└─────────────────────────────────────────────────────────┘
                            ↑
                            │ (when discussion ends)
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
        │   ╔═══════════════════════════════╗   │
        │   ║     DISCUSSION (n iterations) ║   │
        │   ╠═══════════════════════════════╣   │
        │   ║                               ║   │
        │   ║  ┌─────────┐     ┌─────────┐  ║   │
        │   ║  │ GPT ET  │ ←─→ │ Claude  │  ║   │
        │   ║  │(Strateg)│     │ Opus ET │  ║   │
        │   ║  └─────────┘     │ CHAT 1  │  ║   │
        │   ║                  └─────────┘  ║   │
        │   ║       ↻ repeat until          ║   │
        │   ║         moderator cuts        ║   │
        │   ║                               ║   │
        │   ║  Both DO NOT code!            ║   │
        │   ╚═══════════════════════════════╝   │
        │                                       │
        └───────────────────────────────────────┘
                            │
                 conclusions ↓
                            │
                ┌───────────────────┐
                │  Claude Opus      │
                │  (Implementer)    │
                │     CHAT 2        │
                │                   │
                │  ONLY codes!      │
                └───────────────────┘
                            │
                            ↓
                  ┌───────────────────┐
                  │  THIS DOCUMENT   │
                  │ (Source of Truth)│
                  └───────────────────┘
```

### Instances and Their Roles

```
CHAT 1: Claude Opus ET     →  Discussion with GPT ET (no code!) — can be n iterations
CHAT 2: Claude Opus/Sonnet →  Implementation (no discussion!)
CHAT 3: Claude Opus        →  Synthesis (optional, alternative to Gemini)
```

**Chat 2 rule:**
- ONLY implementation — exception: 1–3 short questions allowed if assumptions are blocking
- If no answer → implementer chooses the safest option and clearly states assumptions in output

### Phases

1. **Definition** — Clearly articulate the problem/question
2. **Exploration** — GPT ET explores options, approaches, methodologies
3. **Debate** — Claude ET (Chat 1) and GPT ET exchange perspectives
4. **Synthesis** — *(Optional)* Gemini OR Claude (Chat 3) summarizes conclusions
5. **Implementation** — Claude (Chat 2) executes what was agreed — ONLY code/writing
6. **Review** — GPT ET reviews the result
7. **Iteration** — Repeat as needed
8. **Documentation** — Conclusions go into this file

---

## Working Rules

### Moderator Control

**Discussion duration:** Discussion continues as long as the moderator judges the value of additional iteration exceeds the cost.

**Signals for cutting (guidelines, not mandatory):**
- Arguments start repeating
- Only 1–2 unknowns remain
- Decision is reversible — better to test than debate
- Already enough information for the next step

### Cut (Moderator Cut)

Standard action when moderator decides to end discussion:

1. Summarize current agreement in 3–7 bullets
2. List open points (max 3)
3. Define next step: test/spike/mini-experiment or implementation
4. Send to implementer (Claude Chat 2)

### Context Management
- [ ] Each session starts with a summary of previous state
- [ ] Conclusions are IMMEDIATELY recorded in this document
- [ ] Gemini holds full history for reference (if used)
- [ ] When a model "forgets", consult Gemini or this file

**Without Gemini?** "Source of Truth" + "Pinned Context" are mandatory, synthesis goes to Claude Chat 3 or manually.

**Pinned Context (copy/paste at the start of each new session):**
```
CURRENT STATE:
• Active topic: 
• Last decision (DEC-###): 
• Open questions: 
• Next step: 
• Constraints: 
```

### Discussion Quality
- [ ] Each model gets a chance to give opinion
- [ ] Disagreements are explicitly recorded
- [ ] Decisions include rationale (not just "what" but also "why")
- [ ] Trade-offs are documented

**Conflict Resolver (when GPT and Claude disagree):**
Moderator chooses one of 3 actions:
1. **Cut** — decision now, move on
2. **Test/Spike** — proof in code or measurement
3. **Synthesis** — if confusing, third party summarizes

### Hygiene Habits
- [ ] Notepad++ for preparing longer messages
- [ ] Clean MD file regularly (archive old sessions)
- [ ] Mark what is DECIDED vs what is still OPEN

---

## Templates

### Implementation Brief (for Claude Chat 2)

```
IMPLEMENTATION BRIEF:
**Goal:** (1 sentence)

**Scope (in):**
- 

**Out of scope (don't touch):**
- 

**Acceptance criteria (must pass):**
- [ ] 
- [ ] 

**Constraints:**
- (e.g., no migrations, no breaking API change, single file, etc.)

**Assumptions (if you must assume):**
- 
- ⚠️ If assumptions are not OK → ask moderator before implementation

**Files/Modules (if known):**
- 

**Test/Verification:**
- (steps to prove it works)
```

### Cut (Decision Template)

```
CUT (date):
• What was decided: 
• Why: 
• Open: 
• Next step: 
• Who executes: (Claude Chat 2 / GPT review / me)
```

### Review Output (what GPT returns after review)

```
REVIEW:
**STOP-SHIP issues (if any):**
- 

**Risky assumptions:**
- 

**Quick wins (max 5):**
- 

**Diff/patch suggestions (if any):**
- 

**Verified by:** (compile/run/tests? yes/no)
**How verified:** 
```

### New Topic Template

```
### Topic: [NAME]
**Status:** 🟡 In progress | 🟢 Done | 🔴 Blocked
**Type:** Development | Research | Academic | Strategy
**Context:** [1-2 sentences]
```

---

## Open Questions (Backlog)

Things that weren't cut, but are saved for later.

| # | Question | Why important | How to resolve |
|---|----------|---------------|----------------|
| 1 | | | test / decision / research |
| 2 | | | |
| 3 | | | |

---

## Active Topics

### Topic 1: [NAME]
**Status:** 🟡 In progress | 🟢 Done | 🔴 Blocked

**Context:**
> Brief description of the problem/question

**Discussion:**

| Date | Model | Position/Input |
|------|-------|----------------|
| | GPT | |
| | Claude | |
| | Gemini | |

**Open questions:**
- [ ] 

**Decisions:**
- 

---

## Decision Archive

> `DEC-###` is sequential (DEC-001, DEC-002…) and is never deleted.
> If a decision is superseded, mark as "superseded by DEC-00X".

### [Date] — [Title]
**Decision ID:** DEC-###
**Type:** Development | Research | Academic | Strategy

**Problem:**
> 

**Options considered:**
1. 
2. 
3. 

**Decision:**
> 

**Rationale:**
> 

**Dissenting opinions (if any):**
> 

---

## Lessons Learned

What worked, what didn't — for future reference.

| Date | Situation | Lesson |
|------|-----------|--------|
| | | |

---

## Useful Prompts

### For GPT ET (strategic analysis)
```
Analyze the following problem from the perspective of [domain].
Consider: options, trade-offs, risks, long-term implications.
Don't give a solution — give a framework for thinking.
```

### For Claude ET — Chat 1 (debate, NO coding)
```
We're discussing [topic]. This is a strategic discussion — no coding.
GPT ET proposed: [copy GPT response]
Give your perspective, challenge assumptions, suggest alternatives.
```

### For Claude — Chat 2 (implementation, NO discussion)
```
Based on the following conclusions, implement [X].
Conclusions: [copy from this file]
Focus on: [specific aspect]
Only code/execution — except 1–3 short questions to clarify assumptions if needed.
```

### For Gemini or Claude Chat 3 (synthesis — optional)
```
I have the following discussion between Claude ET and GPT ET.
Summarize key points, identify agreements and disagreements,
and suggest what remains unclear.
```

### For starting a new session
```
Continuing work on [topic].
Current state: [copy from this file]
Today's focus: [specific goal]
```

---

## Meta

**Created:** [Date]
**Last modified:** [Date]
**Version:** 1.2.1 (final)

> Next version (v1.3) only after 2–3 real tasks.
