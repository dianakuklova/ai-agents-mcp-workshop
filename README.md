# AI Agents, MCP & Real Work

> How will coding look in the future? Build it today. By the end of this workshop your AI coding assistant won't just *suggest* code — it will read issues, open pull requests, and drive a browser like a real user. Not a shortcut. A teammate.

A hands-on workshop on **agentic coding** with [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) and the **Model Context Protocol (MCP)**. You'll connect MCP servers to give your agent real context, build a custom skill, and run agentic workflows that actually get things done. The concepts transfer to GitHub Copilot, Cursor, and any other agent-capable tool.

## Start here → the interactive guide

Open **[`workshop-guide.html`](workshop-guide.html)** in your browser and step through it at your own pace. Theory, every command ready to copy, and checkpoints you can tick off. Missed something live? Catch up here.

> If GitHub Pages is enabled for this repo, the guide is also available online — see the link in the repo's **About** section.

## What's inside

| File | What it is |
|------|------------|
| `workshop-guide.html` | Self-guided interactive companion — theory + all modules + cheat sheet |
| `index.html` | The small demo app you build in Module 1 (created during the workshop) |
| `.mcp.json` | MCP server config (GitHub + Playwright) — tokens via env vars |
| `.claude/commands/grill-me.md` | The `/grill-me` custom slash command |

## Prerequisites

Have these ready **before** the workshop:

- **Node.js 18+** — check with `node --version`
- **Claude Code** installed and signed in — `claude --version`, then `claude`
  - Requires a paid plan (Pro / Max / Team / Enterprise) or API credits. The free plan can't run Claude Code.
- A **GitHub account** and a clean test repo with 1–2 open issues
- A **GitHub Personal Access Token (PAT)** with `repo` / `issues` / `pull requests` scope
- **Playwright browsers** — `npx playwright install`

See the setup guide for step-by-step instructions, including the "closest to free" route (Console trial credits) and the VS Code option.

## Quick start

```bash
# 1. Install Claude Code
npm install -g @anthropic-ai/claude-code
claude --version

# 2. Set your GitHub token (add to ~/.zshrc or ~/.bashrc)
export GITHUB_PAT=ghp_xxx

# 3. Clone this repo and launch Claude Code inside it
git clone <this-repo-url>
cd <repo>
claude

# 4. Confirm the MCP servers are connected (inside Claude Code)
/mcp
```

## The modules

0. **Intro & theory** — the agent loop, Claude Code, and what MCP is
1. **Build a demo app** — give the agent something real to work on
2. **`/grill-me`** — a custom skill that refuses fuzzy task definitions
3. **GitHub MCP** — the agent reads issues, opens PRs, comments on reviews
4. **Playwright MCP** — the agent clicks through the app like a real user
5. **Where it breaks** — debugging, limits, and recovering with git

## MCP configuration

The servers are defined in `.mcp.json` at the repo root. The token is read from the `GITHUB_PAT` environment variable — **never hardcode it**.

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": { "Authorization": "Bearer ${GITHUB_PAT}" }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

## Links

- [Claude Code docs](https://docs.claude.com/en/docs/claude-code/overview)
- [Model Context Protocol](https://modelcontextprotocol.io)
- [GitHub MCP server](https://github.com/github/github-mcp-server)
- [Playwright MCP](https://github.com/microsoft/playwright-mcp)

---

*This workshop and its materials move fast — Claude Code and the MCP ecosystem change often. Verify current versions and syntax against the official docs linked above.*
