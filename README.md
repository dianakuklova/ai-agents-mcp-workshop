# AI Agents, MCP & Real Work — Hands-on Assignment

> How will coding look in the future? Today you'll build it. By the end of this session your AI coding assistant won't just *suggest* code — it will read issues, open pull requests, drive a browser like a real user, and refuse to let you start work that isn't properly defined. Not a shortcut. A teammate.
>
> We use **Claude Code** as our primary tool — Anthropic's CLI that brings the full agent loop into your terminal. The same MCP servers and agentic patterns transfer to GitHub Copilot, Cursor, and any other agent-capable tool you already use. Bring your laptop. We build something real.

---

## Before you start — prerequisites

Make sure these are ready (see the setup guide if anything is missing):

- [ ] Node.js 18+ installed (`node --version`)
- [ ] Claude Code installed and authenticated (`claude --version`, then `claude`)
- [ ] A clean test GitHub repo you can push to, with 1–2 open issues
- [ ] GitHub Personal Access Token available as `GITHUB_PAT`
- [ ] Playwright browsers installed (`npx playwright install`)

---

## Module 1 — Build something to work on

Before agents can do real work, they need something real to act on. Start by having Claude Code generate a tiny app — deliberately simple, but with an actual interaction a browser agent can later click through.

**Task:** In Claude Code, run this prompt:

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

**Checkpoint:**
- [ ] `index.html` exists and opens in a browser
- [ ] The form submits and shows the confirmation message
- [ ] The `id` attributes are present (Playwright will need them in Module 4)

---

## Module 2 — A rubber duck that bites back (`/grill-me`)

Agentic workflows fall apart when the task itself is fuzzy. Your first custom skill is a relentless interviewer that refuses to let you move forward until the task is properly defined, edge cases are covered, and requirements are nailed down.

**Task:** Create a custom slash command at `.claude/commands/grill-me.md` that instructs the agent to:
- ask questions one at a time, not all at once
- refuse to move on until inputs, outputs, edge cases, and acceptance criteria are clear
- summarize the final, agreed task definition at the end

**Try it:** Run `/grill-me` and describe a vague feature (e.g. "add search to the page"). Let it interrogate you.

**Checkpoint:**
- [ ] `/grill-me` actually pushes back and doesn't accept a half-baked task
- [ ] It produces a clear task definition at the end

> Note: the exact path/format for skills vs. slash commands evolves — check the current docs: https://docs.claude.com/en/docs/claude-code/overview

---

## Module 3 — Wire up the GitHub MCP server

MCP (Model Context Protocol) is an open standard that gives your agent real context about your environment. Now connect the GitHub server and watch the agent work the development loop without leaving the terminal.

**Task:** Add the GitHub MCP server (remote HTTP; the old npm package is deprecated):

```bash
claude mcp add --scope project --transport http github \
  https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer ${GITHUB_PAT}"
```

Verify with `/mcp` — it should show `github` as connected.

**Then have the agent run the full loop:**
1. Read the open issues in your test repo.
2. Pick one and implement a small change (e.g. tweak the page from Module 1).
3. Create a pull request for the change.
4. Add a comment on the code review.

**Checkpoint:**
- [ ] Agent successfully read issues from the repo
- [ ] Agent created a PR
- [ ] Agent commented on a review — all without you touching the GitHub UI

---

## Module 4 — Playwright MCP: use the app like a real user

"Works on my machine" ends here. The Playwright server lets the agent navigate your app like a real user — clicking through flows, taking screenshots, catching what only shows up when someone actually uses the thing.

**Setup:**
```bash
claude mcp add --scope project playwright -- npx -y @playwright/mcp@latest
```
Verify with `/mcp`.

Work through these three passes in order.

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

**Checkpoint:**
- [ ] Agent navigated the page and produced screenshots
- [ ] Agent detected the error-state behavior on its own
- [ ] Agent produced a QA report

> Tip: add `--save-video` to the Playwright server to record the whole run as a video.

---

## Module 5 — Where it breaks

Real-world adoption means understanding the limits, not just the possibilities. Now break things on purpose and learn to debug them.

**Task:** Trigger at least two of the following and recover from each:
- [ ] `command not found: npx` — a Node / PATH problem (common inside WSL)
- [ ] MCP server won't start — a typo in config, or `GITHUB_PAT` not set
- [ ] GitHub PAT lacking permissions — the agent can't create a PR
- [ ] Playwright without installed browsers — fails on navigation
- [ ] The agent loops or does something destructive — how do you stop it (Esc) and roll back with git?

For each: write down **how you recognized it** (the error) and **how you fixed it**.

---

## Wrap-up — what you've built

By the end you've turned a plain coding assistant into something that can reason through a problem, make decisions, and execute across your whole toolchain: it defined the task with you (`/grill-me`), worked the GitHub loop, exercised the app like a user (Playwright), and you know what to do when it misbehaves.

The same patterns transfer to Figma, Postgres, Jira, and the fast-growing ecosystem of other MCP servers — and to Copilot, Cursor, and any other agent-capable tool you already use. This isn't just another feature. It's an infrastructural shift in how AI tools work.

### Stretch goals (if you have time)
- [ ] Wire up a third MCP server relevant to your own stack (Postgres, Jira, Figma…)
- [ ] Have `/grill-me` feed its task definition straight into a GitHub issue
- [ ] Chain it end to end: define → implement → PR → browser-test → report, in one session

