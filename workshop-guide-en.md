# AI Agents, MCP & Real Work — Workshop Guide

> Turn an assistant that *suggests* code into one that *acts*. Today you'll build an AI agent that drives a browser like a real user — not a shortcut, a teammate. We use **Claude Code** as our primary tool; the concepts transfer to GitHub Copilot, Cursor, and any other agent-capable tool.

---

## Intro & theory

### Agentic coding and the agent loop

A classic assistant **suggests** code — you accept or reject it. An agentic assistant gets a **goal** and decides the steps itself. It runs in a loop:

```
goal → think → use a tool → check the result → iterate → done
```

Say "fix the login bug" and the agent **reads** the files, **finds** the problem, **edits** the code, **runs** the tests, and if they fail it tries again — without you dictating each step.

### Claude Code

Our primary tool. It brings that agent loop right into your terminal (or VS Code). It has real access to your files, can run commands, and connects to external tools via MCP. It runs on **Node.js 18+** and a **paid plan** or API credits — the free plan won't run it.

### MCP — Model Context Protocol

Before, every connection between an AI and a tool (GitHub, a database, Figma…) had to be built separately. MCP is an **open standard** that gives AI apps and tools one shared language.

> **Analogy:** MCP is like **USB-C for AI** — one connector you plug anything into, instead of a different cable for every device.

- **Host** — the app you work in (Claude Code).
- **Server** — a program that exposes capabilities (GitHub, Playwright…).
- **Tools / Resources / Prompts** — actions the model calls · data it can read · ready-made templates.

MCP isn't just another feature — it's an infrastructural shift. The server ecosystem is growing fast: GitHub, Figma, Postgres, Jira, Playwright.

---

## Prerequisites

Check that this is all working. If anything's missing, see the setup guide or ask the instructor.

- [ ] **Node.js 18+** (check with `node --version`)
- [ ] **Claude Code** installed and authenticated (`claude --version`, then `claude`)
- [ ] A clean test **GitHub repo** with 1–2 open issues
- [ ] A **GitHub PAT** available as `GITHUB_PAT`
- [ ] **Playwright browsers** (`npx playwright install`)

---

## Build something to work on

The agent needs something real. Have Claude Code generate a tiny page with an interaction that Playwright will later click through.

> The `id` attributes are there on purpose — Playwright needs them to click reliably.

**Paste this prompt into Claude Code:**

```
Create a simple static web page as a single index.html file with the CSS inline in a <style> tag. No frameworks, no extra JavaScript beyond what's needed for the interaction below. Goal: it should serve as a demo that Playwright MCP will later open and click through.

Page content:
- An <h1> heading with the text "Workshop Demo".
- A short paragraph describing the page.
- A simple contact form with fields:
  - Name (input, id="name")
  - Email (input type email, id="email")
  - Message (textarea, id="message")
  - A "Submit" button (id="submit")
- When Submit is clicked, show a confirmation message (without a page reload, via a small bit of JS) in an element with id="confirmation", e.g. "Thank you, your message has been sent."

Design:
- Clean, minimalist CSS: centered content, a max-width container, readable font, subtle borders and padding on the form.
- No external resources (no CDN, fonts, or images) — everything offline in a single file.

At the end, tell me how to open/run the page locally.
```

**Checkpoint**
- [ ] `index.html` exists and opens in a browser
- [ ] The form submits and shows the confirmation message
- [ ] The `id` attributes are present

---

## Playwright MCP — use the app like a real user

"Works on my machine" ends here. The agent navigates your app like a person — clicking, taking screenshots, catching what only shows up in real use.

**Add the server:**

```
claude mcp add --scope project playwright -- npx -y @playwright/mcp@latest
```

Verify with `/mcp`. Work through the three passes in order.

### 4a — Happy path

```
Open the index.html file in the browser via Playwright. Fill in the form with test data:
name "John Smith", email "john@example.com", message "This is a test message".
Click the Submit button (id="submit"). Verify that the confirmation message appears
in the element with id="confirmation". Take a screenshot of the resulting page and tell me what you saw.
```

### 4b — Error state

```
Open index.html via Playwright. Try to submit the form in ways that test error states:
1. Leave the email field empty and click Submit — check how the page behaves.
2. Enter an invalid email (e.g. "abc") and click Submit — check whether the browser or the
   page shows a validation error.
After each attempt take a screenshot and describe what happened. At the end, tell me whether
the page behaves as it should and what could be improved.
```

### 4c — Full QA pass

```
Take index.html and run a full QA pass via Playwright as a real user would:
1. Open the page and take a screenshot of the initial state.
2. Check that all elements are present: heading, form, name/email/message fields, Submit button.
3. Test the happy path — fill in valid data, submit, verify the confirmation message, screenshot.
4. Test the error state — empty or invalid email, verify the behavior, screenshot.
5. Give me a short report: what works, what doesn't, and a list of issues found with recommendations.
Work step by step and after each step tell me what you did.
```

> Tip: add `--save-video` to the Playwright server to record the whole run as a video.

**Checkpoint**
- [ ] Agent navigated the page and produced screenshots
- [ ] Agent detected the error-state behavior on its own
- [ ] Agent produced a QA report

---

## Every command in one place

**Setup**

```
node --version
npm install -g @anthropic-ai/claude-code
claude --version
claude
```

**GitHub PAT**

```
export GITHUB_PAT=github_pat_your_token
```

**MCP servers**

```
claude mcp add --scope project --transport http github \
  https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer ${GITHUB_PAT}"

claude mcp add --scope project playwright -- npx -y @playwright/mcp@latest

npx playwright install
```

**In Claude Code**

```
/mcp        # list servers and their status
Esc         # stop a running agent
```

**Git rescue**

```
git status
git checkout -- .      # discard local changes
git reset --hard HEAD  # hard reset (careful!)
```

**.mcp.json** (in the repo root; tokens via env)

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

---

## Questions

**Is it free?**
The account and install are. Using Claude Code isn't — you need a paid plan (Pro from ~$20/mo) or API credits. Closest to free is a Console account with trial credits (~$5, no card), good for a few sessions.

**Terminal or VS Code?**
Either — it's the same Claude Code. The VS Code extension (publisher Anthropic, verified checkmark) gives you a panel in the editor; the CLI is a reliable fallback. Both need the CLI under the hood and a paid plan.

**Is it safe to give the agent GitHub access?**
Use a PAT with minimal scope, on a test repo. Never hardcode the token into files — use `GITHUB_PAT`. You can revoke it anytime on GitHub.

**What if the agent does something dumb?**
That's why we work on a test repo and use git. Stop it with `Esc`. Roll back with `git checkout` / `git reset`.

**Does this work with Copilot / Cursor?**
Yes. MCP is an open standard, so the same servers and patterns run across agent-capable tools. The surface changes, not the principle.

### Quick troubleshooting

| Symptom | Fix |
|---|---|
| `command not found: npx` | Install Node 18+; in WSL install Node inside WSL |
| Claude Code won't authenticate | Check paid plan / credits; sign in again |
| Server not "connected" in `/mcp` | Check config, `echo $GITHUB_PAT`, restart the session |
| Agent can't create a PR | PAT with repo/issues/PR scope; set it again |
| Playwright fails on navigation | `npx playwright install` |
| Agent loops | `Esc`; roll back with git; make the task more specific |

---

*This moves fast — verify current versions in the [Claude Code docs](https://docs.claude.com/en/docs/claude-code/overview) and at [modelcontextprotocol.io](https://modelcontextprotocol.io).*
