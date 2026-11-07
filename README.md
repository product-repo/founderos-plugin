# FounderOS plugin for Claude

Work on your [FounderOS](https://founderos.productrepo.ai) checklists with Claude. Claude reads a checklist, does the work, saves the results and evidence in FounderOS, and marks each step done.

The plugin adds:

- **FounderOS connector**: the remote MCP server at `https://founderos.productrepo.ai/mcp/founder`. You sign in with your FounderOS account. Claude acts only in the workspace you pick, with your role.
- **FounderOS checklist runner** skill: the protocol Claude follows to work a checklist step by step.

You need a FounderOS account with at least one product.

## Contents

- [Install in the Claude app (claude.ai, Claude Desktop, Cowork)](#install-in-the-claude-app)
- [Install in Claude Code (terminal, Desktop Code tab, VS Code, JetBrains)](#install-in-claude-code)
- [Sign in to FounderOS](#sign-in-to-founderos)
- [Run your first checklist](#run-your-first-checklist)
- [Update the plugin](#update-the-plugin)
- [Troubleshooting](#troubleshooting)
- [Uninstall](#uninstall)
- [Release a new version](#release-a-new-version) (maintainers)

## Install in the Claude app

For claude.ai on the web, Claude Desktop (chat) and Cowork.

1. Open **Customize → Plugins**.
2. Click **Add → Add marketplace**.
3. Enter `product-repo/founderos-plugin`. Turn on **Sync automatically**, so you get new versions. Click **Add**.
4. Find **FounderOS** in the list and click **Install**.
5. Open **Settings → Capabilities** and turn on **Code execution and file creation**. Skills need it.

Then [sign in to FounderOS](#sign-in-to-founderos).

> On a Team or Enterprise plan, an owner can limit plugins and connectors. If you cannot add the marketplace or connect, ask an owner.

## Install in Claude Code

For the `claude` terminal app, the Code tab in Claude Desktop, and the VS Code and JetBrains extensions. Use one of these three ways.

### Option 1: Ask Claude

Paste this prompt into Claude Code:

```text
Install the FounderOS plugin for me. Run `claude plugin install founder-os --marketplace product-repo/founderos-plugin` in the terminal. If that option is unknown, run `claude plugin marketplace add product-repo/founderos-plugin` and then `claude plugin install founder-os@founderos`. When done, tell me to run /reload-plugins and then /mcp to sign in to FounderOS.
```

### Option 2: Slash commands

Type these in a Claude Code session, one at a time:

```text
/plugin marketplace add product-repo/founderos-plugin
/plugin install founder-os@founderos
/reload-plugins
```

Claude Code 2.1.275 or later can do it in one command:

```text
/plugin install founder-os --marketplace product-repo/founderos-plugin
```

### Option 3: Terminal

Run this in your shell (Claude Code 2.1.292 or later):

```bash
claude plugin install founder-os --marketplace product-repo/founderos-plugin
```

On an older version, run two commands:

```bash
claude plugin marketplace add product-repo/founderos-plugin
claude plugin install founder-os@founderos
```

The plugin installs for your user, in all projects. To install it for one project only, add `--scope project` (shared with the repo) or `--scope local` (only you).

Start a new session, or run `/reload-plugins` in an open one. Then [sign in to FounderOS](#sign-in-to-founderos).

## Sign in to FounderOS

The connector uses OAuth. You sign in once per Claude app.

**Claude app:** open the FounderOS plugin and go to its **Connectors** tab. Click **Connect** on `founder-os`. Sign in to FounderOS and pick the workspace Claude may use.

**Claude Code:** run `/mcp`. Pick `plugin:founder-os:founder-os` and choose **Authenticate**. Your browser opens FounderOS. Sign in and pick the workspace.

You can see and revoke connections in FounderOS under **Settings → Connect Claude**.

## Run your first checklist

1. In FounderOS, open a product, then open a checklist.
2. On a step, click **Open in Claude**. Or click **Copy prompt** and paste it in a new Claude chat or Claude Code session.
3. Send it. Claude reads the checklist, loads its skills and starts the first step.
4. Claude saves results and evidence in FounderOS and marks each step done.
5. When a step needs approval, review it in FounderOS. Then tell Claude to continue.

Claude asks you before anything it cannot undo, such as publish, send, pay or delete.

## Update the plugin

**Claude app:** with **Sync automatically** on, new versions arrive by themselves.

**Claude Code:** auto-update is off for this marketplace by default. Turn it on in `/plugin` → **Marketplaces** → `founderos` → **Enable auto-update**. Or update by hand:

```bash
claude plugin marketplace update founderos
claude plugin update founder-os@founderos
```

Then run `/reload-plugins` or start a new session.

## Troubleshooting

**FounderOS tools do not show.** Claude app: start a new chat, open the tools menu and turn FounderOS on. Claude Code: run `/reload-plugins`, then `/mcp` to check the connector status.

**Two FounderOS connectors or skills.** Remove the old ones. Keep only the plugin.

- Claude app: remove an old custom connector named FounderOS in **Settings → Connectors**, and an old uploaded skill in **Settings → Capabilities → Skills**.
- If you install the plugin in the Claude app and also in Claude Code, Claude Code can show it two times (`founder-os@synced` and `founder-os@founderos`). Uninstall one: `claude plugin uninstall founder-os@founderos`.

**`Permission denied (publickey)` when you add the marketplace.** You have no GitHub SSH key. Use HTTPS:

```bash
export CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1
claude plugin marketplace add product-repo/founderos-plugin
```

Or add it with the full URL: `https://github.com/product-repo/founderos-plugin.git`.

**`unknown option '--marketplace'`.** Your Claude Code is too old for the one-step install. Run `claude update`, or use the two-step commands.

**Claude asks you to connect again.** Your sign-in expired or was revoked. Claude app: click **Connect** on the plugin's Connectors tab. Claude Code: run `/mcp` and authenticate again.

**Wrong workspace.** Revoke the connection in FounderOS under **Settings → Connect Claude**. Then connect again and pick the right workspace.

**Still stuck?** Open an [issue](https://github.com/product-repo/founderos-plugin/issues).

## Uninstall

**Claude app:** open **Customize → Plugins**, open FounderOS and uninstall it.

**Claude Code:**

```bash
claude plugin uninstall founder-os@founderos
claude plugin marketplace remove founderos
```

## Release a new version

This repo is the plugin's source. There is no CI. Release by hand.

1. Copy the changed skills from the FounderOS app (`resources/skills/<skill>/`). Copy only skills without `in_plugin: false` in their metadata. Keep each folder name the same as the skill `name`.
2. Bump `version` in `.claude-plugin/plugin.json` (semver). Set it there only, not in `marketplace.json`. Users get an update only when this value changes.
3. Validate:

   ```bash
   claude plugin validate --strict .
   ```

4. Test locally:

   ```bash
   claude plugin marketplace add ./
   claude plugin install founder-os@founderos
   ```

5. Commit, push to `main`, then tag the release:

   ```bash
   claude plugin tag --push
   ```

### Layout

```text
.claude-plugin/
  marketplace.json   # marketplace "founderos", one plugin at the repo root
  plugin.json        # plugin "founder-os": name, version, metadata
  icon.png
.mcp.json            # the FounderOS connector (remote MCP over HTTP)
skills/
  founder-os-checklist-runner/SKILL.md
```
