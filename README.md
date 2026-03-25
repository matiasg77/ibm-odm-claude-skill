# 🧠 IBM ODM Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that teaches Claude how to **read, understand, create, and modify** IBM Operational Decision Manager (ODM) project artifacts.

## What it does

When this skill is active, Claude Code understands:

- **Project structure** — How ODM rule projects are organized (rules, BOM, ruleflows, deployment configs)
- **BAL rules** (`.brl`) — Business Action Language syntax, conditions, actions, definitions
- **Decision tables** (`.dta`) — Partition-based tables with range syntax and multi-dimensional conditions
- **Ruleflows** (`.rfl`) — Execution flow orchestration with tasks, transitions, and branching
- **BOM/XOM mapping** (`.bom`) — Business Object Model to Java Execution Object Model relationships
- **Deployment configs** (`.dep`) — RuleApp packaging for Rule Execution Server (RES)
- **Debugging** — Common issues like unresolved BOM members, UUID conflicts, deployment failures

## Supported ODM versions

- IBM ODM 8.9.x
- IBM ODM 8.10.x
- IBM ODM 8.11+ (including container/K8s deployments)

## Installation

### Option 1: Install from `.skill` file

```bash
claude install-skill ibm-odm.skill
```

### Option 2: Manual installation

Copy the `ibm-odm/` folder to your Claude Code skills directory:

```bash
# Global installation
cp -r ibm-odm ~/.claude/skills/

# Or project-level installation
cp -r ibm-odm .claude/skills/
```

## Skill structure

```
ibm-odm/
├── SKILL.md                      # Main skill instructions
└── references/
    └── version-notes.md          # ODM version differences (8.9 → 8.11+)
```

## Usage examples

Once installed, Claude Code automatically activates this skill when you work with ODM projects. Try prompts like:

- *"Read through this ODM project and explain the rule logic"*
- *"Create a new BAL rule that checks if the customer's credit score is above 700"*
- *"Add a new condition column to this decision table"*
- *"Explain the execution flow in this ruleflow"*
- *"Create a BOM entry for a new Java class `com.example.model.Account`"*
- *"Debug why this rule isn't firing — here's the .brl file"*

## Who is this for?

- **ODM developers** who want AI-assisted rule development
- **Business analysts** working with Decision Center who need help understanding rule logic
- **Teams migrating** ODM projects between versions
- **Anyone new to ODM** who needs to understand an existing rule project

## Contributing

PRs welcome! Some ideas for expanding the skill:

- Additional reference files for specific industries (credit scoring, insurance underwriting, healthcare)
- BAL pattern library with common rule templates
- Simulation data provider examples
- Decision Center REST API reference

## License

MIT


