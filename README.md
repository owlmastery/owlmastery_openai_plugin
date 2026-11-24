# Owl Mastery for Codex

A Codex plugin and marketplace that connect Codex to the Owl Mastery MCP server. The server gives Codex read-only access to Owl Mastery's daily gamma levels, implied volatility and market data. The plugin adds two skills that teach Codex how to read that data.

**Status.** The server answers at `https://owlmastery.com/mcp` once Owl Mastery routes it. Until then, test against a local API (see [Test against localhost](#test-against-localhost)).

## Install

Add the marketplace, then install the plugin:

```bash
codex plugin marketplace add owlmastery/owlmastery_openai_plugin
codex plugin add owlmastery --marketplace owlmastery
```

To install from a local checkout, run `codex plugin marketplace add .` from the repository root. Restart Codex after you install.

## Create and set an API key

The server accepts a bearer API key from a registered Owl Mastery user. A key starts with `owl_`. You create it while signed in, and it works for 14 days. After that, calls answer 401 and you create a new key.

Set the key in the `OWLMASTERY_API_KEY` environment variable of the shell that starts Codex:

```bash
export OWLMASTERY_API_KEY=owl_...
```

Never write the key into a file in this repository or into a Codex config file. Both plugins read the variable by name. After you change the variable, restart Codex.

## What the plugin adds

- **MCP server `owlmastery`**, streamable HTTP, with `bearer_token_env_var = OWLMASTERY_API_KEY`. Codex reads the variable when it connects. `codex mcp list` shows it under Bearer Token Env Var.
- **Skill `levels`**. Reads a ticker's call resistance, put support, HVL, gamma wall, net gamma and implied volatility, in a fixed tool order.
- **Skill `market`**. Summarizes breadth, indexes, movers, sectors and your watchlists.

The server has seven read-only tools: `search_tickers`, `get_ticker_levels`, `get_gamma_exposure`, `get_volatility`, `get_market_overview`, `get_sectors` and `list_watchlists`. The server describes their inputs itself, so the skills name the tools and leave the schemas out.

## Test against localhost

The bundled server URL is fixed. To test against a local API on `http://localhost:8080`, add a second server and turn the bundled one off:

```bash
codex mcp add owlmastery-local --url http://localhost:8080/mcp --bearer-token-env-var OWLMASTERY_API_KEY
```

```toml
# ~/.codex/config.toml
[plugins."owlmastery@owlmastery".mcp_servers.owlmastery]
enabled = false
```

Remove both when you are done. If you skip the plugin and want only the server, run the `codex mcp add` command against `https://owlmastery.com/mcp` and use the skills from this repository by hand.

## What works where

| Client | Works now | Notes |
| --- | --- | --- |
| Codex CLI and desktop app | Yes | Bearer token from an environment variable. |
| ChatGPT web connectors | Not yet | They offer OAuth or no authentication only, and the server has no OAuth yet. |
| Claude Code | See the Claude plugin repository | |

A public listing in the plugin directory needs a different authentication path than an environment variable. Raise it with OpenAI before you submit.

## Keep the skills in sync

The skill files in `plugins/owlmastery/skills/` are the same text as in the Claude Code plugin repository. This repository holds a copy; the source of truth is the Claude plugin repository. Edit the skills there first, then copy `skills/` to the other repository unchanged. The skill text has no client-specific wording, so the copy needs no edits.

## Layout

```text
.agents/plugins/marketplace.json    the marketplace, named owlmastery
plugins/owlmastery/
  .codex-plugin/plugin.json         the plugin manifest
  .mcp.json                         the MCP server and its bearer variable
  skills/levels/                    SKILL.md and references/glossary.md
  skills/market/                    SKILL.md
```

This plugin uses the Codex compatibility layout (`.codex-plugin/plugin.json` and `.mcp.json`), because only that form can name a bearer environment variable.

The data is educational information, not investment advice.
