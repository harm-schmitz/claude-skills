# claude-skills

A team marketplace of [Claude Code](https://docs.claude.com/en/docs/claude-code) plugins and skills.

Add the marketplace once, then install any plugin from it. Right now it ships:

| Plugin | What it gives you |
| --- | --- |
| `design-docs-plugin` | The `design-docs` skill — grills you one question at a time to capture design decisions, then writes a full `docs/` folder (overview, data model, Mermaid flow diagram, ADRs) *before* you write any code. |

## Prerequisites: the Claude Code CLI

Plugins are installed with the `/plugin` command, which **only works in the interactive terminal CLI** (`claude` in a terminal). The VS Code / JetBrains extensions run a bundled copy of Claude Code but do **not** expose the `/plugin` command — so even if you use the IDE extension day to day, you need the CLI installed to add this marketplace.

Check whether you already have it:

```
claude --version
```

If that says "not found", install it. The **native installer is recommended** (it auto-updates in the background) — pick the line for your shell:

- **Windows PowerShell:**
  ```
  irm https://claude.ai/install.ps1 | iex
  ```
- **Windows CMD:**
  ```
  curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
  ```
- **macOS / Linux / WSL:**
  ```
  curl -fsSL https://claude.ai/install.sh | bash
  ```

Alternatives: `winget install Anthropic.ClaudeCode` on Windows, or npm (`npm install -g @anthropic-ai/claude-code`) if you already have [Node.js](https://nodejs.org) 22+.

If `claude` still isn't found right after installing, close and reopen your terminal so it picks up the updated PATH. Then verify with `claude --version`. See the [official install docs](https://code.claude.com/docs/en/setup) if you hit trouble.

> **Windows PATH warning after `irm` install?** If the installer prints *"Native installation exists but `C:\Users\<you>\.local\bin` is not in your PATH"*, the install worked — that folder just isn't on your PATH yet, so `claude` won't be found. Add it once in PowerShell, then open a **new** terminal:
> ```powershell
> [Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";$env:USERPROFILE\.local\bin", "User")
> ```
> (Or add the path via System Properties → Environment Variables → Edit User PATH, as the installer message describes.)

Then sign in once:

```
claude
```

You also need access to this GitHub repo: `harm-schmitz/claude-skills`.

## Install the plugin

Start the interactive CLI by running `claude` in a terminal, then type these at the prompt.

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
