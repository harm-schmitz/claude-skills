# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **plugin marketplace** (`harm-schmitz/claude-skills`) — no application code, no build/test/lint tooling. It's a thin registry (`.claude-plugin/marketplace.json`) pointing at one or more plugin folders under `plugins/`, each of which bundles skills (and potentially commands/agents/hooks, though none exist yet).

There is nothing to build, lint, or test here. "Development" means editing Markdown skill files and JSON manifests, then verifying they load correctly via the `/plugin` command in an interactive `claude` session.

## Repo structure

```
.claude-plugin/marketplace.json           # marketplace manifest — lists all plugins
plugins/<plugin-name>/
├── .claude-plugin/plugin.json            # plugin manifest (name, version, description, author)
├── skills/<skill-name>/SKILL.md          # skill definition: YAML frontmatter (name, description) + instructions body
├── skills/<skill-name>/assets/           # templates/static files the skill copies or fills in when it runs
├── commands/  agents/  hooks/            # optional, not currently used by any plugin
```

The marketplace manifest's `plugins[].source` is a relative path to the plugin folder; `plugins[].name` must match the plugin's own `.claude-plugin/plugin.json` `name`. Installed as `<plugin-name>@<marketplace-name>` (e.g. `design-docs-plugin@claude-skills`).

## Adding or editing a plugin

1. Add `plugins/<your-plugin>/.claude-plugin/plugin.json` (name, version, description, author) and a `skills/` (or `commands/`/`agents`/`hooks/`) folder inside it.
2. Register the plugin in the top-level `.claude-plugin/marketplace.json` `plugins` array — `name` must match step 1's manifest exactly.
3. For a new skill, write `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`) followed by the instructions body. The `description` is what triggers the skill automatically, so write it to include concrete phrases a user would say.
4. Any files a skill writes into a user's project (templates, boilerplate) go in `skills/<skill-name>/assets/` and are referenced from the SKILL.md body — they are not executed, just copied/filled in by Claude at skill-invocation time.
5. **Plugins are distributed in isolation**: on install, Claude Code copies only that plugin's own directory into the user's plugin cache — files elsewhere in this repo (a repo-root file, or another plugin's folder) are never copied and won't resolve, even though relative paths appear to work when testing locally from a full clone of this repo. A relative reference is only safe if the target file lives *inside the same plugin's* directory tree. To share content across plugins, package it as its own skill (with a broad, standalone trigger description) and have other skills invoke it by name via the Skill tool instead of reading its files directly — that's how `language-guide-plugin` is shared with `design-docs-plugin` below.

## Verifying changes

There's no automated test suite. To check a change actually works, install/reload the marketplace from a local path in an interactive `claude` session:

```
/plugin marketplace add /path/to/this/repo
/plugin install <plugin-name>@claude-skills
```

Restart Claude Code if prompted, then trigger the skill (by its natural-language description or its explicit `/<skill-name>` command) and confirm the behavior matches the SKILL.md instructions.

## Existing plugin: design-docs-plugin

Ships the `design-docs` skill: grills the user one question at a time to capture design decisions for a new (or existing, documented retroactively) project, then writes a `docs/` folder into the *user's* project (not this repo) — `docs/design/overview.md`, `docs/design/data_model.md`, `docs/design/flows/main_flow.md` (Mermaid), and `docs/decisions/NNN_<title>.md` ADR files. The skill's own file (`SKILL.md`) documents the exact question order, output structure, and the templates in `assets/` to use — read it directly rather than duplicating that detail here if you need to modify the skill's behavior.

Key behaviors baked into the skill (preserve these if editing SKILL.md):
- Reads the target codebase first and only asks the user what the code can't already answer.
- Grills one question at a time (never a batch of questions) and offers a suggested answer where possible, working through a fixed section order: Problem → Users/stories → Entities → Data flow → Flow design → Boundaries → Open questions → Decisions.
- If `docs/` already exists in the target project, it diffs and updates in place rather than regenerating, and continues ADR numbering from the highest existing number instead of restarting at 001.
- Only records a decision as an ADR when it's hard to reverse, non-obvious, or the result of a real tradeoff — not every implementation detail (target: 3-6 ADRs per session).
- Writes files straight to disk with the Write tool — never presents doc content as copyable markdown blocks.

The `description` field in SKILL.md frontmatter is what makes Claude Code auto-invoke the skill (in addition to the explicit `/design-docs` command) — it must contain concrete phrases a user would actually say (e.g. "I want to build X", "let's design X"), not an abstract summary of what the skill does. Keep this in mind for any new skill added to this marketplace, not just this one.

`design-docs`'s step 3 (writing the docs) invokes the `language-guide` skill before writing prose — see below.

## Existing plugin: language-guide-plugin

Ships the `language-guide` skill: a shared writing-style guide (target B2-level English) for any documentation prose, not just this marketplace's own output. It triggers standalone (e.g. "write a wiki page", "review this doc for clarity") and is also invoked by `design-docs` before it writes `overview.md`, `data_model.md`, or ADRs.

The guide's content — the detection heuristics and the examples log — lives entirely inside `skills/language-guide/SKILL.md`, not in a separate file. That's deliberate: this skill's whole job is to inject that content into context when triggered, so there's no benefit to a separate asset file, and a separate file outside the plugin's own directory wouldn't survive distribution anyway (see "Adding or editing a plugin" above).

When a remark about wording surfaces during other work (e.g. while running `design-docs`), add it to the examples log in `skills/language-guide/SKILL.md` under the matching heuristic — don't start a new working-notes file elsewhere; this SKILL.md is the single source of truth.
