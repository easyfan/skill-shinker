# skill-shrinker

Shrink bloated Claude Code skill/agent/command files into a clean three-layer structure.

## What it does

Analyzes a target SKILL.md (or command/agent .md) and compresses it using a four-class
extraction system (ABCD), with two target tracks depending on skill complexity:

**ABCD extraction classes:**

| Class | Destination | Criteria |
|-------|-------------|----------|
| A | stays in SKILL.md | orchestration logic, flow steps, branch decisions |
| B | `scripts/` | bash blocks >3 lines, callable standalone |
| C | `manifest-schema.json` | cross-phase shared parameters consumed by >1 Phase |
| D | `DESIGN.md` | design rationale, derivations, background (not loaded into context) |

**Two target tracks:**

| Track | Target | When |
|-------|--------|------|
| Option A — Stable | SKILL.md ≤ 220 lines | single-phase or stable skill |
| Option B — Growth | SKILL.md ≤ 80 lines (pure coordinator) | ≥3 Phases, growing complexity |

Option B is **Proposal-only**: skill-shrinker outputs the agent decomposition plan and
directory structure; the actual split is performed manually.

**Three operating modes based on file size:**

| Lines | Action |
|-------|--------|
| < 200 | Inform user: no compression needed |
| 200–500 | Output a **Proposal** (ABCD classification + estimated result) — does not modify files |
| > 500 | Full analysis, user confirms A or B track, then executes |

## Used by skill-review

skill-review (v1.5.0+) enforces a **400-line hard gate**: any target file exceeding 400 lines is
rejected with instructions to run `/skill-shrink` first. Install skill-shrinker alongside
skill-review to enable reviewing larger skill/agent files.

```
⛔ skill-review refuses to review files > 400 lines.
   Run /skill-shrink <file> first, then retry.
```

## Installation

### Option 1: Marketplace (recommended)

```
/plugin marketplace add skill-shrinker
/plugin install skill-shrinker@latest
```

### Option 2: Manual install

```bash
bash install.sh
```

Or with a custom Claude config directory (`CLAUDE_DIR`):

```bash
bash install.sh --target=/path/to/.claude
# or: CLAUDE_DIR=/path/to/.claude bash install.sh
```

Preview without writing:

```bash
bash install.sh --dry-run
```

Uninstall:

```bash
bash install.sh --uninstall
```

## Usage

After installation, trigger in Claude Code:

```
/skill-shrink my-skill
shrink ~/.claude/skills/my-skill/SKILL.md
this skill is getting too big, can you split it
```

skill-review will also guide you here automatically when a target file exceeds 400 lines.

## Output structure

**Option A (Stable):**

```
my-skill/
├── SKILL.md              ← ≤220 lines, orchestration only
├── manifest-schema.json  ← cross-phase shared parameters (written once by init phase)
├── scripts/              ← extracted bash (each callable standalone)
│   └── init_something.sh
└── DESIGN.md             ← design notes, derivations, background
```

**Option B (Growth, Proposal-only):**

```
my-skill/
├── SKILL.md              ← ≤80 lines, pure coordinator
├── manifest-schema.json  ← cross-phase shared parameters
├── agents/               ← one file per Phase
│   ├── phase-0-init.md
│   └── phase-1-xxx.md
└── DESIGN.md
```

## Evals

The `evals/evals.json` file contains 5 test cases covering the key decision paths:

| ID | Scenario | Expected behavior |
|----|----------|-------------------|
| 1 | Large skill (>500 lines, multi-phase) | ABCD analysis + Option B recommendation when Phase ≥ 3 |
| 2 | Medium skill (200–500 lines) | Proposal mode: ABCD classification, await y/n/B confirmation |
| 3 | Small skill (<200 lines) | Reject: inform user no compression needed |
| 4 | Growth-type skill (4 Phases) | Option B Proposal-only: agent decomposition plan + manifest schema |
| 5 | C vs D boundary (LUFS target + derivation) | LUFS → C class (manifest); derivation formula → D class (DESIGN.md) |

## Changelog

### v0.3.0 (2026-04-20)

Major design revision — ABCD four-class extraction system + dual-track Option A/B:

| Item | Change |
|------|--------|
| Classification | ABC three-class → ABCD four-class; C split into C (manifest) and D (DESIGN.md) |
| Manifest | New `manifest-schema.json` for cross-phase shared parameters; prevents context drift |
| Option B | Growth track: Phase agent decomposition (Proposal-only), SKILL.md ≤ 80 lines |
| Decision tree | Two-step C vs D tree with boundary case table |
| Error handling | File existence check, invalid input handling, syntax check failure warning |
| Evals | 5 test cases covering all decision paths |

### v0.2.0 (2026-04-14)

skill-review integration — skill-shrinker is now a required companion for skill-review v1.5.0+:

| Item | Change |
|------|--------|
| Dependency role | skill-review v1.5.0 enforces a 400-line hard gate and instructs users to run `/skill-shrink` first |
| Post-install notice | install.sh now detects skill-review and confirms the companion relationship |
| README | Added "Used by skill-review" section |

### v0.1.0 (2026-03-01)

Initial release.

## License

MIT
