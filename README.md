# Backend Design Skills

[English](./README.md) | [中文](./docs/lang/README.zh.md)

A Claude Code skill that brings expert, principle-driven guidance for backend
system design directly into your coding sessions. Whenever you ask about
architecture, API design, service decomposition, domain modeling, or code
structure, Claude consults this skill before answering — so the advice you get
is grounded in widely accepted industry principles rather than ad-hoc opinions.

## What It Does

The `backend-design` skill teaches Claude to apply the following **Golden
Principles** to your backend questions:

| Principle | Scope | Core Idea |
|---|---|---|
| S.O.L.I.D | Class / Module | OOP design rules for maintainable code |
| DRY | Any level | Eliminate duplication |
| KISS | Any level | Prefer simplicity |
| YAGNI | Feature level | Don't build what you don't need yet |
| Clean Architecture | Application | Decouple business logic from infrastructure |
| Domain-Driven Design | Domain | Model software around the business domain |
| Microservices | System | Decompose by business capability |
| 12-Factor App | Deployment | Build portable, scalable cloud services |

Deep-dive reference material for S.O.L.I.D, DDD, Microservices, Clean
Architecture, and API Design lives under
`skills/backend-design/references/` and is loaded on demand.

### When It Activates

The skill is automatically consulted when your question touches on:

- "How should I structure my project / services?"
- "Is this good design?" / "What pattern should I use here?"
- API design, service boundaries, domain modeling
- Refactoring for maintainability
- Any backend architecture or system-design question

## Installation

### Option 1 — Install via Plugin Marketplace (Recommended)

Inside Claude Code, add this repository as a marketplace and install the
plugin:

```
/plugin marketplace add bhrajate/backend-design-skills
/plugin install backend-design@backend-design-skills
```

Other useful commands:

```
/plugin marketplace list          # list installed marketplaces
/plugin marketplace update        # pull latest plugin definitions
/plugin marketplace remove <name> # remove a marketplace
```

### Option 2 — Install from a Local Clone

```bash
git clone git@github.com:bhrajate/backend-design-skills.git backend-design-skills
```

Then in Claude Code:

```
/plugin marketplace add ./backend-design-skills
/plugin install backend-design@backend-design-skills
```

### Option 3 — Manual Install (User-level Skill)

Copy the skill directory into your user-level skills folder:

```bash
mkdir -p ~/.claude/skills
cp -r skills/backend-design ~/.claude/skills/
```

Claude Code discovers skills under `~/.claude/skills/<name>/SKILL.md`
automatically — restart your session and the skill is live.

## Verifying the Installation

After installing, ask Claude a design question such as:

> "How should I split my monolith into services?"

Claude should reference the `backend-design` skill and ground its answer in the
relevant principles (YAGNI, DDD bounded contexts, etc.).

## Repository Layout

```
.
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest
│   └── marketplace.json     # Marketplace manifest
├── skills/
│   └── backend-design/
│       ├── SKILL.md         # Skill entry point
│       └── references/      # Deep-dive docs loaded on demand
│           ├── solid.md
│           ├── ddd.md
│           ├── microservices.md
│           ├── clean-architecture.md
│           └── api-design.md
└── docs/
    ├── assets/
    └── lang/
        └── README.zh.md
```

## License

See the `LICENSE` file at the root of the repository.
