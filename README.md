# claude-skills

A team marketplace of [Claude Code](https://docs.claude.com/en/docs/claude-code) plugins and skills.

Add the marketplace once, then install any plugin from it. Right now it ships:

| Plugin | What it gives you |
| --- | --- |
| `design-docs-plugin` | The `design-docs` skill — grills you one question at a time to capture design decisions, then writes a full `docs/` folder (overview, data model, Mermaid flow diagram, ADRs) *before* you write any code. |

## Prerequisites

- Claude Code installed and signed in (`claude` in your terminal, or the VS Code / JetBrains extension).
- Access to this GitHub repo: `harm-schmitz/claude-skills`.

## Install

Run these inside a Claude Code session (type them at the prompt).

**1. Add this marketplace** (only needed once per machine):

```
/plugin marketplace add harm-schmitz/claude-skills
```

`harm-schmitz/claude-skills` is GitHub `owner/repo` shorthand. If you cloned the repo locally instead, you can point at the folder: `/plugin marketplace add /path/to/claude-skills`.

**2. Install the plugin:**

```
/plugin install design-docs-plugin@claude-skills
```

The `@claude-skills` suffix is the marketplace name (from `.claude-plugin/marketplace.json`), not the repo name — they happen to match here.

**3. Restart Claude Code** if prompted, so the skill loads.

That's it. You can also just run `/plugin`, browse to the `claude-skills` marketplace, and install interactively if you prefer a menu over typing.

## Use it

Once installed, the `design-docs` skill activates automatically at the start of a new project or feature — say something like *"I want to build X"* or *"let's design a new flow"*. You can also invoke it explicitly:

```
/design-docs
```

## Update

Pull the latest plugin versions from the marketplace with:

```
/plugin marketplace update claude-skills
```

## Uninstall / remove

```
/plugin uninstall design-docs-plugin@claude-skills
/plugin marketplace remove claude-skills
```

## Optional: auto-install for the whole team

Instead of everyone running the commands above, you can commit the marketplace + plugin into a project's `.claude/settings.json` so anyone who opens that repo gets prompted to trust and install it automatically:

```json
{
  "extraKnownMarketplaces": {
    "claude-skills": {
      "source": {
        "source": "github",
        "repo": "harm-schmitz/claude-skills"
      }
    }
  },
  "enabledPlugins": {
    "design-docs-plugin@claude-skills": true
  }
}
```

## Contributing a new plugin or skill

1. Add a plugin folder under `plugins/<your-plugin>/` with a `.claude-plugin/plugin.json` manifest and your `skills/`, `commands/`, `agents/`, or `hooks/` inside it.
2. Register it in the top-level [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) `plugins` array.
3. Open a PR. Once merged, teammates get it via `/plugin marketplace update claude-skills`.

See the [Claude Code plugins docs](https://docs.claude.com/en/docs/claude-code/plugins) and [plugin marketplaces docs](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for manifest details.
