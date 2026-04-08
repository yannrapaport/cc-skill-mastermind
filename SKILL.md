---
name: mastermind
description: Multi-agent review — launch parallel expert agents (PM, Architect, Dev, etc.) to analyze a complex topic, plan, codebase, or document from multiple perspectives and produce a synthesis. Use when the user asks for a review, second opinion, or sanity check on something non-trivial; or proactively before committing to an important architectural/strategic decision.
---

# Mastermind

Launch parallel expert agents to review a topic, codebase, plan, or document from multiple perspectives, then synthesize their findings.

## Arguments

- `$ARGUMENTS` — the subject to review (a description, file path, or directory)
- Optional flag: `--agents role1,role2,role3` to override default agents and skip auto-detection

## Available roles

| Role | Persona | Style | Focus |
|------|---------|-------|-------|
| CEO | Steve Jobs | Visionary, demanding, "think different" | Strategic direction, product vision, ruthless prioritization |
| PM | Lenny Rachitsky | Pragmatic, growth-oriented, accessible | User value, scope, UX, user stories |
| Designer | Jony Ive | Simplicity, obsession with detail, form follows function | UX/UI, design coherence, simplicity |
| Dev | DHH | Efficient, modern, no overengineering, "less is more" | Implementation, edge cases, testability, cost |
| Reviewer | Dirty Harry (Clint Eastwood) | Strict, direct, no compromise | Critical flaws, risks, what could go wrong |
| Architect | John Carmack | Obsessed with elegance, performance, zero hidden debt | Structure, patterns, scalability, feasibility |
| Documentation | Richard Feynman | If you can't explain it simply, you don't understand it | Clarity, completeness, consistency |
| Security | Bruce Schneier | Threat modeling, pragmatic, never alarmist | Vulnerabilities, data, auth, compliance |
| Project Manager | Andy Grove | OKRs, no-excuse delivery, "Only the Paranoid Survive" | Planning, risks, dependencies, delivery |

**Defaults:** CEO, PM, Dev, Architect, Reviewer

**Contextuals (via `--agents` or auto-detection):** Designer, Documentation, Security, Project Manager

---

## Step 0 — Auto-detect roles (if no `--agents` flag)

Analyze the subject to infer the most relevant roles. Use these heuristics:

| Subject type | Roles |
|-------------|-------|
| Product / strategy / vision | CEO + PM + Reviewer |
| Technical / architecture / infra | Architect + Dev + Reviewer |
| Spec / proposal / documentation | PM + Documentation + Reviewer |
| UX / design / user flow | Designer + PM + CEO |
| Organisation / process / team | CEO + PM + Project Manager |
| Security / compliance | Security + Architect + Dev |
| Ambiguous or broad | CEO + PM + Dev + Architect + Reviewer (full defaults) |

State which roles were selected and why (one line) before proceeding.

---

## Step 1 — Parse input

Extract the subject from `$ARGUMENTS`. If `--agents` is provided, use those roles and skip Step 0.

---

## Step 2 — Prepare workspace

Create the directory `mastermind-{today's date YYYY-MM-DD}/` inside a `scratchpad/` folder in the current working directory. If it already exists (re-run), increment a counter suffix until a free name is found: `mastermind-YYYY-MM-DD-2/`, `mastermind-YYYY-MM-DD-3/`, etc.

Note: the `scratchpad/` folder is created in the current working directory if it doesn't exist.

---

## Step 3 — Launch agents in parallel

For each selected role, launch an Agent (subagent_type: general-purpose) **in parallel**.

Each agent receives this prompt (adapt {role}, {persona}, {style}, {focus}, {subject}):

```
You are a {role} expert in a multi-agent review exercise ("mastermind").
You embody **{persona}** — {style}. Let this personality come through in your tone, phrasing, and judgments.

**Subject to analyze:**
{subject}

**Your focus:** {focus}

**Your mission:**
1. If the subject is a file path or directory, read it first
2. Analyze the subject through the lens of your expertise
3. Identify strengths, weaknesses, and concrete proposals

**Write your analysis to:** {workspace}/{role-slug}.md

**Required format:**
# {role} ({persona})

## Strengths
- ...

## Weaknesses
- ...

## Proposals
- ...
```

---

## Step 4 — Synthesize

Once all agents complete, launch one final Agent to synthesize:

```
You are the synthesizer in a mastermind exercise.

Read ALL files in {workspace}/ (each expert's analysis).

Produce a synthesis at {workspace}/synthesis.md using this format:

# Mastermind Synthesis — {subject}

## Verdict
One direct sentence: yes / no / yes with conditions.

## Consensus
Points on which all experts agree.

## Debates
Points of disagreement or tension between analyses.

## Recommended actions
Prioritized list of concrete actions, integrating the best proposals from each expert. Distinguish "blockers" (must be done first) from "polish" (nice to have).

## Agents consulted
- {role} ({persona}): 1-line summary of their position
```

---

## Step 5 — Present results

Show the user:
1. Which roles were selected (and why, if auto-detected)
2. A brief summary of each agent's key finding (1 line each)
3. The path to synthesis.md
4. Ask: "Want me to dig deeper on any point, or shall we act on the recommendations?"
