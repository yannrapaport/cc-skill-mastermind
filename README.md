# mastermind

A Claude Code skill that launches parallel expert agents to review any topic — codebase, plan, spec, decision — from multiple perspectives, then synthesizes their findings into a single actionable report.

## How it works

1. You provide a subject (description, file path, or directory)
2. Claude selects the most relevant expert roles based on the subject type (or you pick them via `--agents`)
3. Each expert agent runs in parallel, writing its analysis to a shared workspace folder
4. A synthesis agent reads all analyses and produces a final report with verdict, consensus, debates, and prioritized recommendations

## Requirements

- [Claude Code](https://claude.ai/code) CLI with parallel agent support (Sonnet or Opus tier recommended)

## Installation

```bash
git clone https://github.com/yannrapaport/cc-skill-mastermind ~/.claude/skills/mastermind
```

To verify the install worked, run `/my-skills` in Claude Code — `mastermind` should appear in the list.

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

If omitted, roles are auto-detected from the subject type: technical subjects get `Architect + Dev + Reviewer`, product/strategy subjects get `CEO + PM + Reviewer`, broad or ambiguous subjects get all 5 defaults. See [SKILL.md](./SKILL.md) for the full heuristics table.

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

**Default roles (for broad/ambiguous subjects):** CEO, PM, Dev, Architect, Reviewer

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

The `scratchpad/` folder is git-trackable — useful for sharing reviews with your team.

## Example output

The following is an actual synthesis produced by this skill:

```markdown
# Mastermind Synthesis — README.md clarity and actionability

## Verdict

Yes with conditions: the README is functional and its example output is strong, but it
ships with several gaps that will erode trust and generate friction before a first run.

---

## Consensus

- The intro one-liner works — clear mental picture in one sentence.
- The roles table is clean, scannable, and effective.
- No error/failure coverage: what happens when an agent fails or synthesis runs with
  partial inputs? Zero coverage.
- Cost and model implications are inadequately communicated.

---

## Debates

- **Structural reordering**: PM argues for leading with the example output — "here's what
  you get" before "here's how it works." Documentation and Reviewer prefer filling gaps
  in the current structure. Both are valid; depends on whether the goal is adoption or
  documentation.
- **Missing license**: Reviewer flags it as an adoption blocker for cautious engineers.
  Others don't raise it. Probably right for a public repo targeting developers.

---

## Recommended actions

### Blockers
1. Add a verification step to the install flow — users have no way to confirm it worked.
2. Document failure modes: skill not found, agent fails mid-run, partial synthesis.
3. Add a cost/usage callout — spawning 5 Opus agents on a large codebase is not free.

### Polish
- Add a second usage example targeting a code review use case (`./src/auth/`).
- Add a LICENSE file — removes a silent adoption blocker for careful engineers.
- Consider leading with the example output for conversion-optimized structure.

---

## Agents consulted

- **Documentation (Richard Feynman)**: Praised the example output; flagged the missing
  verification step and defaults contradiction as the clearest structural failures.
- **Reviewer (Dirty Harry)**: Most adversarial read — flagged missing license, cost
  blindspot, and vague version requirements as adoption blockers.
- **PM (Lenny Rachitsky)**: Pushed on the missing emotional hook — the README explains
  mechanics but doesn't answer "why should I care?"; proposed leading with example output.
```

