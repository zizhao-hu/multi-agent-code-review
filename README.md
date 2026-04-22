# Commit Review Panel

**Theme:** multi-agent
**One-liner:** A pre-commit hook that spawns three LLM agents—a security skeptic, a readability purist, and a pragmatist—who debate the diff in a structured review before letting you push.

## Problem

Code review happens too late (after the PR) or not at all (solo devs, fast iterations). Pre-commit linters catch syntax but miss design smells, security patterns, or "this will confuse your future self" issues. You need human judgment earlier, but summoning a reviewer for every commit is unrealistic.

## The sketch

Three agents run concurrently on `git diff --staged`. Agent 1 (Security) scans for common vulnerabilities and data leaks. Agent 2 (Readability) flags unclear variable names, missing comments where logic is dense, and overly clever one-liners. Agent 3 (Pragmatist) argues for shipping and calls out when the other two are bikeshedding. Each agent posts a short opinion (2-4 sentences). A fourth orchestrator agent synthesizes the three takes into a single go/no-go recommendation with a confidence score. If score < 0.6, the commit is blocked and you see the full debate transcript. If >= 0.6, it logs the debate but lets you through.

## Why now

Structured outputs (JSON mode) make the debate protocol trivial to parse. Parallel LLM calls are fast enough (~2-3s for three agents) that this doesn't break flow. Devs are used to pre-commit hooks that take a few seconds (Prettier, ESLint). The cost per commit is negligible with cheaper models (Haiku, GPT-4o-mini).

## Demo surface

A terminal recording: `git commit -m "feat: add user export"` triggers the hook, three agent summaries print in columns, the orchestrator verdict appears, and the commit either proceeds or aborts with a colorized transcript.

## Risks / honest take

The agents might agree too often (rubber-stamping) or disagree on trivial stuff (alert fatigue). Tuning the prompts so they actually argue without degenerating into nitpicks is the hard part. Also, devs will disable the hook the first time it blocks a "quick fix" at 11pm.

## Stack guess

Python (git hook script), `anthropic` or `openai` SDK for parallel calls, `rich` for terminal formatting, Claude Haiku or GPT-4o-mini for the three agents, a slightly stronger model (Sonnet) for the orchestrator.

---

_Spawned from [auto-brainstorm](https://github.com/zizhao-hu/auto-brainstorm) on 2026-04-22. Theme: `multi-agent`._
