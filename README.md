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
# Mastermind Synthesis — API design ready for v1?

## Verdict
Not yet — two structural blockers before shipping.

## Consensus
- Error responses are inconsistent across endpoints
- No pagination on list endpoints

## Debates
- Dev thinks cursor pagination is overkill at this scale; Architect disagrees

## Recommended actions
### Blockers
1. Standardize error envelope: { error: { code, message } } everywhere
2. Add cursor-based pagination to /users and /events

### Polish
- Add request ID to all responses for traceability
- Document rate limits in OpenAPI spec
```

## Cost note

Each run spawns 3–5 parallel agents plus one synthesis agent. Expect roughly 6–10 API calls per `/mastermind` invocation. On Sonnet, this is fast (under 2 minutes) but not free.
