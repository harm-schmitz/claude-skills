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

> **`Command 'git' not found or is in an unsafe location (current directory)`?** This bites Windows users who installed Git for Windows as a *per-user* install (git.exe under `C:\Users\<you>\AppData\Local\Programs\Git\...`) **and** started the CLI from their home folder. Claude Code refuses to run a `git` that resolves from inside the current directory, and your home folder is an ancestor of that git install, so a legitimate git looks "unsafe." Git itself is fine — `git --version` still works. Two fixes:
> - **Quick:** start the CLI from an actual project folder instead of your home directory (e.g. `cd C:\Users\<you>\Documents\projects`), then re-run the `/plugin marketplace add` command. Any real subfolder works because the AppData git path isn't underneath it.
> - **Permanent:** reinstall Git for Windows **system-wide** so git.exe lives at `C:\Program Files\Git\cmd\git.exe`, outside your user tree. Then it's never "under" a home-based working directory.

**2. Install the plugin:**

```
/plugin install design-docs-plugin@claude-skills
```

The `@claude-skills` suffix is the marketplace name (from `.claude-plugin/marketplace.json`), not the repo name — they happen to match here.

**3. Restart Claude Code** if prompted, so the skill loads.

The folder you run these commands from does **not** scope where the skill works — it only matters for satisfying the git check above. Marketplaces and installed plugins are recorded in your user-global config (`~/.claude`), so once installed the skill is available in **every** project on that machine, no matter which directory you installed it from.

That's it. You can also just run `/plugin`, browse to the `claude-skills` marketplace, and install interactively if you prefer a menu over typing.

## Use it

Once installed, the `design-docs` skill activates automatically at the start of a new project or feature — say something like *"I want to build X"* or *"let's design a new flow"*. You can also invoke it explicitly:

```
/design-docs
```

## Update

This marketplace tracks **latest**: plugins here carry no pinned version, so each is identified by its current git commit. Whenever a change is pushed to `harm-schmitz/claude-skills`, that becomes the new version automatically — there's no version number to bump.

To pull the newest commit manually, run both:

```
/plugin marketplace update claude-skills
/plugin update design-docs-plugin@claude-skills
```

The first refreshes the marketplace catalog from GitHub; the second installs the latest commit of the plugin.

Better: enable background auto-updates so you never have to. See [auto-install for the whole team](#optional-auto-install-for-the-whole-team) below — with `"autoUpdate": true` set, Claude Code pulls new commits on its own (shortly after startup, applied at the next launch).

## How plugins and updates work under the hood

When you add and install a plugin, Claude Code keeps **two separate local copies** under `~/.claude/plugins/` (your user-global config — not the project folder you ran the command from):

- **The marketplace clone** — `marketplaces/claude-skills/` is a real `git clone` of this GitHub repo. It's the *source*.
- **The installed plugin** — `cache/claude-skills/design-docs-plugin/<version>/` is a plain, flat copy of just that plugin's files, materialized from the clone when you install or update. **This copy is what actually runs.**

**Invoking the skill never touches GitHub.** `/design-docs` loads from the local cached copy — it works offline and pulls nothing. The network is only used during three git operations, each a fetch/pull on the marketplace clone:

| Action | What it does |
| --- | --- |
| `/plugin marketplace add` | `git clone` the repo into `marketplaces/` |
| `/plugin marketplace update` | `git pull` the clone — refreshes the *source*, but **not** the running copy |
| background auto-update | the same `git pull`, on a timer a few minutes after startup |

**Getting a new version = re-copying the cache from the freshly pulled clone.** That is what `/plugin update` (and auto-update) does. This is the reason `marketplace update` *alone* isn't enough: it advances the clone to the latest commit, but the cached copy the skill runs from stays frozen until an install/update re-materializes it. So either run both commands, or let `autoUpdate` handle it.

Because this marketplace pins no version (see [Contributing](#contributing-a-new-plugin-or-skill)), a plugin's identity **is** its git commit. Every push to the default branch is a new "version," so "latest" always means the newest commit on GitHub as of the last pull.

## Uninstall / remove

```
/plugin uninstall design-docs-plugin@claude-skills
/plugin marketplace remove claude-skills
```

## Optional: auto-install for the whole team

Instead of everyone running the commands above, you can commit the marketplace + plugin into the global or a project's `.claude/settings.json` so anyone who opens that repo gets prompted to trust and install it automatically. With `"autoUpdate": true`, teammates also stay on the latest commit without ever running an update command:

```json
{
  "extraKnownMarketplaces": {
    "claude-skills": {
      "source": {
        "source": "github",
        "repo": "harm-schmitz/claude-skills"
      },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "design-docs-plugin@claude-skills": true
  }
}
```

> Auto-update runs in the background a short, random delay after Claude Code starts (up to ~10 minutes) and applies at the next launch — so it's automatic, but not instantaneous. Third-party marketplaces have auto-update **off** by default, which is why the `autoUpdate` flag above is what turns it on.

### Which `settings.json` to put it in

The exact same block works at either scope — pick based on who you want it to cover:

| File | Scope | Use when |
| --- | --- | --- |
| `<repo>/.claude/settings.json` | **Project** — checked into a repo, applies to anyone who opens it | You want your whole team to get the plugin automatically |
| `~/.claude/settings.json` | **User-global** — applies to every project on *your* machine | You just want it installed everywhere for yourself, no per-repo setup |

If both define it, the project file wins for that repo (project settings override user settings). There's also `~/.claude/settings.local.json` for machine-specific overrides you don't want to commit. So a common personal setup is simply to drop the block into your global `~/.claude/settings.json` once and never think about it again — you'll get the plugin (and, with `autoUpdate`, the latest commit) in every project.

## Contributing a new plugin or skill

1. Add a plugin folder under `plugins/<your-plugin>/` with a `.claude-plugin/plugin.json` manifest and your `skills/`, `commands/`, `agents/`, or `hooks/` inside it.
2. Register it in the top-level [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) `plugins` array.
3. Open a PR. Once merged, teammates get it via `/plugin marketplace update claude-skills` (or automatically, if they have `autoUpdate` on).

> **Do not add a `version` field** to `plugin.json` or to the marketplace entry. This marketplace intentionally tracks latest by git commit — a pinned `version` freezes users on the cached copy until it's manually bumped, so new commits would silently fail to reach anyone.

See the [Claude Code plugins docs](https://docs.claude.com/en/docs/claude-code/plugins) and [plugin marketplaces docs](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for manifest details.
