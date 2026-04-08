# mastermind

A Claude Code skill that launches parallel expert agents to review any topic — codebase, plan, spec, decision — from multiple perspectives, then synthesizes their findings into a single actionable report.

## How it works

1. You provide a subject (description, file path, or directory)
2. Claude selects the most relevant expert roles based on the subject type (or you pick them via `--agents`)
3. Each expert agent runs in parallel, writing its analysis to a shared workspace folder
4. A synthesis agent reads all analyses and produces a final report with verdict, consensus, debates, and prioritized recommendations

## Requirements

- [Claude Code](https://claude.ai/code) CLI (any recent version)
- Runs best on Sonnet or Opus tier models (each review spawns 3–5 parallel agents)

## Installation

```bash
git clone https://github.com/yannrapaport/cc-skill-mastermind ~/.claude/skills/mastermind
```

No restart needed — Claude Code picks up new skills automatically.

## Usage

```
/mastermind <subject>
/mastermind <subject> --agents Role1,Role2,Role3
```

**Examples:**

```
/mastermind Is my API design ready for v1?
/mastermind ./src/auth/
/mastermind Should we migrate to a monorepo?
/mastermind ./docs/spec.md --agents Security,Architect,Dev
```

### `--agents` flag

Override auto-detection with a comma-separated list of role names (case-insensitive, no spaces):

```
--agents CEO,PM,Reviewer
--agents documentation,security
```

Valid values: `CEO`, `PM`, `Designer`, `Dev`, `Reviewer`, `Architect`, `Documentation`, `Security`, `ProjectManager`

If omitted, roles are inferred from the subject: technical subjects get `Architect + Dev + Reviewer`, product/strategy subjects get `CEO + PM + Reviewer`, etc. See `SKILL.md` for the full heuristics table.

## Available roles

| Role | Persona | Focus |
|------|---------|-------|
| CEO | Steve Jobs | Strategic direction, vision, prioritization |
| PM | Lenny Rachitsky | User value, scope, UX |
| Designer | Jony Ive | UX/UI, design coherence, simplicity |
| Dev | DHH | Implementation, edge cases, testability |
| Reviewer | Dirty Harry | Critical flaws, risks, what could go wrong |
| Architect | John Carmack | Structure, patterns, scalability |
| Documentation | Richard Feynman | Clarity, completeness, consistency |
| Security | Bruce Schneier | Vulnerabilities, data, auth, compliance |
| ProjectManager | Andy Grove | Planning, risks, dependencies, delivery |

**Default roles (when no `--agents` flag):** CEO, PM, Dev, Architect, Reviewer

> **Note:** The personas listed are fictional AI characters inspired by real public figures. This skill is not affiliated with or endorsed by any of the individuals mentioned.

## Output

Each run creates a timestamped workspace inside `scratchpad/` in your current working directory (created if it doesn't exist). Re-runs on the same day increment a counter:

```
scratchpad/
├── mastermind-YYYY-MM-DD/
├── mastermind-YYYY-MM-DD-2/
└── mastermind-YYYY-MM-DD-3/
    ├── dev.md
    ├── architect.md
    ├── reviewer.md
    └── synthesis.md   ← start here
```

## Example output (synthesis excerpt)

```markdown
# Mastermind Synthesis — Should we migrate to a monorepo?

## Verdict
No — not yet. The case isn't strong enough to justify the migration cost at current team size.

---

## Consensus

- The current multi-repo setup is causing real friction (shared library versioning, cross-repo PRs).
- A monorepo would solve the coordination problem but introduce tooling complexity the team hasn't dealt with before.
- The decision hinges on team size and CI/CD maturity, not on the monorepo concept itself.

---

## Debates

- **Timing**: Dev argues the team is too small to absorb the tooling overhead now; Architect
  disagrees — earlier adoption means less migration debt later. Lean toward Dev's position
  given current headcount.
- **Tooling choice**: CEO wants to avoid Nx/Turborepo lock-in; Architect considers them
  non-negotiable for a monorepo at scale. Worth a spike before committing.

---

## Recommended actions

### Blockers
1. Define the trigger criteria: at what team size / repo count does migration become worth it?
2. Run a 1-week spike with one shared library extracted into a candidate monorepo — measure
   actual CI impact before deciding.

### Polish
- Document the current cross-repo dependency map to make the coordination cost visible.
- Evaluate Turborepo vs. Nx on a throwaway branch (2 days max).

---

## Agents consulted

- **CEO (Steve Jobs)**: Skeptical of tooling complexity — wants a crisp "why now" before committing.
- **PM (Lenny Rachitsky)**: Supports migration if it unblocks the shared design system work.
- **Dev (DHH)**: Against it at current scale — the cure is worse than the disease right now.
- **Architect (John Carmack)**: Pro-migration but only with proper tooling; warns against DIY solutions.
- **Reviewer (Dirty Harry)**: Called out the lack of any cost/rollback analysis as a red flag.
```

