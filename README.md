# Flotiq Agent Skills

Agent Skills that teach AI coding assistants how to work with [Flotiq](https://flotiq.com) — the API-first headless CMS — effectively and idiomatically.

This repository does **not** re-publish the [Flotiq documentation](https://flotiq.com/docs/). It teaches workflows, conventions, and judgment calls: how to model content, populate it, consume it from an app, and keep generated types in sync. Whenever an implementation detail is needed (exact request shape, field option, CLI flag), the skills point to the canonical docs instead of copying them — so the skills stay small and don't rot when the API changes.

Inspired by the architecture and philosophy of the [WordPress Agent Skills](https://github.com/WordPress/agent-skills) project.

## Work in progress

This repository is in Proof of concept state.

See next steps:
* [ ] Adjust catalog of flotiq skills
* [ ] Use https://www.skills.sh to manage skills

## Why Agent Skills

AI coding assistants are capable, but without project-specific guidance they tend to:

* invent content type shapes instead of checking what already exists
* hand-write TypeScript interfaces instead of using generated types
* reach for raw `fetch()` instead of the official SDK
* duplicate data across content types instead of using relations
* mix schema design, content population, and frontend integration into one undifferentiated task

Agent Skills fix this by giving the assistant small, single-purpose playbooks it can load only when relevant — "when to use this," "how the workflow should go," "what to avoid," and "how to confirm you got it right" — rather than one large document covering everything.

## Repository structure

```text
flotiq-agent-skills/
├── README.md
└── skills/
    ├── flotiq-schema/     # Design and evolve Content Type Definitions
    │   ├── SKILL.md
    │   └── references/
    ├── flotiq-content/    # Populate projects with Content Objects
    │   ├── SKILL.md
    │   └── references/
    ├── flotiq-sdk/        # Build applications on the official SDK
    │   ├── SKILL.md
    │   └── references/
    ├── flotiq-ui-sections/ # Render Flowbite sections in Flotiq pages
    │   ├── SKILL.md
    │   └── references/
    ├── flotiq-typegen/    # Generate and maintain TypeScript types
    │   ├── SKILL.md
    │   └── references/
    └── flotiq-plugins/    # Build and publish Flotiq UI plugins
        ├── SKILL.md
        └── references/
```

Each skill directory is self-contained:

* **`SKILL.md`** — the skill itself: when to use it, the workflow, best practices, common mistakes, a completion checklist, and links to official documentation.
* **`references/`** — short, focused notes that back up the workflow (naming conventions, field-type cheat sheets, checklists). References summarize; they don't duplicate the Flotiq docs.

There are no scripts in this version. If a skill later needs deterministic helpers (e.g. a schema validator), they'd live in a `scripts/` folder next to `SKILL.md`, following the same pattern WordPress Agent Skills uses.

## Skill philosophy

* **One skill, one question.** `flotiq-schema` answers "how should this content be modeled?" `flotiq-content` answers "what content should exist?" `flotiq-sdk` answers "how does my app read this content?" `flotiq-typegen` answers "how do I keep types in sync?" No skill tries to answer more than one of these.
* **Workflows and judgment, not API reference.** Skills tell an assistant how to think about a task and what "done" looks like. They defer to [flotiq.com/docs](https://flotiq.com/docs/) for exact payloads, endpoints, and field options.
* **Composable.** An assistant only loads the skill relevant to the task at hand — designing a schema doesn't pull in SDK caching advice, and generating types doesn't pull in content-modeling advice.
* **No duplication across skills.** Concepts owned by one skill (e.g. relations are designed in `flotiq-schema`) are referenced, not re-explained, from the others.

## Using these skills

These skills follow the emerging [Agent Skills](https://github.com/WordPress/agent-skills) convention — a `SKILL.md` with YAML frontmatter (`name`, `description`) that an agent uses to decide when to load the rest of the file. This format is understood by Claude Code, GitHub Copilot, Cursor, Codex, and other Agent Skills-compatible tools.

### Install with Skills (recommended)

Run from your project root. The [Skills CLI](https://www.npmjs.com/package/skills) interactively lets you choose your AI assistant and whether to install for the current project or globally:

```sh
npx skills add flotiq/flotiq-skills
```

Install all Flotiq skills for GitHub Copilot in the current project without prompts:

```sh
npx skills add flotiq/flotiq-skills --agent github-copilot --skill '*' --yes
```

Install globally for Claude Code:

```sh
npx skills add flotiq/flotiq-skills --agent claude-code --skill '*' --global --yes
```

Install selected skills or target other agents:

```sh
npx skills add flotiq/flotiq-skills --agent cursor --skill flotiq-sdk --skill flotiq-typegen
```

See the [Skills CLI supported-agent list](https://github.com/vercel-labs/skills#supported-agents) for available agent identifiers and installation paths. The CLI also supports listing, updating, and removing installed skills.

### Manual installation

Copy any skill folder from `skills/` directly into your AI assistant's skills directory.

## Available skills

| Skill | Answers |
|---|---|
| [`flotiq-schema`](skills/flotiq-schema/SKILL.md) | How should I design or evolve this Content Type? |
| [`flotiq-content`](skills/flotiq-content/SKILL.md) | How do I populate this project with good content? |
| [`flotiq-sdk`](skills/flotiq-sdk/SKILL.md) | How do I build an app on top of Flotiq content? |
| [`flotiq-ui-sections`](skills/flotiq-ui-sections/SKILL.md) | How do I render or customize Flotiq page sections? |
| [`flotiq-typegen`](skills/flotiq-typegen/SKILL.md) | How do I generate and maintain TypeScript types? |
| [`flotiq-plugins`](skills/flotiq-plugins/SKILL.md) | How do I build, configure, or publish a Flotiq UI plugin? |

## Adding new skills

The structure is deliberately flat and additive — a new skill is a new folder under `skills/` with its own `SKILL.md` and `references/`. Nothing about the existing skills needs to change to add:

* `flotiq-webhooks` — reacting to content changes
* `flotiq-nextjs` — Next.js-specific integration patterns built on `flotiq-sdk`
* `flotiq-media` — deeper media library / asset transformation workflows
* `flotiq-ai` — AI-assisted content generation workflows

When proposing a new skill, keep it answering exactly one question, and check whether the knowledge it needs already belongs to an existing skill before duplicating it.

## Official documentation

* [Flotiq Documentation](https://flotiq.com/docs/)
* [Flotiq MCP Server](https://mcp.flotiq.com/mcp)
* [Next.js integration](https://flotiq.com/docs/Universe/nextjs/nextjs-setup)
* Type generation: `npm exec flotiq-api-typegen`
