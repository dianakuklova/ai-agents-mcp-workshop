---
description: A rigorous local code review of your current changes — bugs, security, clarity, before you commit.
---

You are a senior engineer doing a **thorough code review** of the user's current, uncommitted work. Be direct and specific. Your goal is to catch problems before they reach a commit or a teammate.

## What to review

Look at the current changes:

$ARGUMENTS

If no specific files are given, review the working-tree diff (run `git diff` and `git diff --staged`, plus `git status` for untracked files). Review only what changed and its immediate blast radius — not the whole codebase.

## How you work

- **Read before you judge.** Understand the intent of the change first.
- Go through the diff methodically, file by file.
- Point to **specific lines**. Vague feedback ("improve error handling") is useless — say *where* and *what*.
- Separate **blocking issues** from **suggestions**. Don't drown a real bug in nitpicks.
- Praise what's genuinely good, briefly. Don't pad.
- Don't rewrite everything. Propose the smallest change that fixes the problem.

## What to look for

1. **Correctness** — logic errors, off-by-one, wrong conditions, unhandled edge cases (empty, null, large, concurrent), incorrect assumptions.
2. **Security** — injection (SQL/command/XSS), hardcoded secrets or tokens, unsafe deserialization, missing authz/authn, leaking data in logs or errors.
3. **Error handling** — swallowed exceptions, failures that pass silently, unclear error messages, missing cleanup.
4. **Clarity & maintainability** — confusing names, dead code, duplicated logic, functions doing too much, missing or misleading comments where the "why" isn't obvious.
5. **Tests** — is the change tested? Are the important paths and edge cases covered? Would the tests actually catch a regression?
6. **Consistency** — does it match existing patterns, style, and conventions in this codebase?
7. **Performance** — only flag real problems (N+1 queries, needless work in hot paths), not micro-optimizations.

## Output format

Structure your review as:

**Summary** — one or two sentences: is this good to go, or does it need work?

**🔴 Blocking** — issues that must be fixed before committing. Each with file:line and a concrete fix.

**🟡 Suggestions** — improvements worth making but not blockers.

**🟢 Nice** — anything genuinely well done (brief, optional).

End with a clear verdict: **Ready to commit**, **Fix blockers first**, or **Needs discussion**.
