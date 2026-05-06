---
layout: post
title: "Claude Code keeps ignoring skills. I built a hook to fix it."
---

I kept reminding Claude it has skills. Claude kept ignoring them.

It's not just me. Skills are supposed to activate on their own when the prompt is relevant, and in practice they often don't.

## Why skills miss

A skill activates when Claude reads its description and decides the prompt matches. That's a judgment call competing with the user's actual question, the conversation history, and `CLAUDE.md`. A polite "Use when creating a pull request…" doesn't fight for attention. It gets passed over and Claude does the thing directly.

## What actually helps

**Write descriptions as directives, not invitations.** The default "Use when…" reads like a suggestion Claude can pass on. Replace it with "ALWAYS invoke this skill when…", list the exact triggers, and add a "do not do X directly — use this skill first" clause that names the tool or action Claude would otherwise reach for. Here's what one of mine looks like after the rewrite:

```yaml
description: Creates or edits a GitHub Pull Request. ALWAYS invoke
  this skill when the user asks to create a PR, open a pull request,
  draft a PR, submit code for review, or edit a PR title or
  description. Do not run `gh pr create` / `gh pr edit` or write a PR
  body directly — use this skill first.
```

Helps, but you have to do it for every skill, and you can't fix descriptions on skills shipped by someone else's plugin.

**Use a `UserPromptSubmit` hook.** Claude Code runs it on every prompt and lets it inject context the model has to read. The hook decides what Claude sees, so descriptions stop being the only signal. Works on any skill, regardless of who wrote the description.

Both work, but the hook does better for me. I built a small plugin around it, [`skill-suggestion`](https://github.com/pavliuko/claude-plugins/tree/main/plugins/skill-suggestion). It scores each prompt against rules you declare per skill in a project-local `rules.json`, then injects a ranked banner of matches into context. Below threshold, no banner — no tax on irrelevant prompts.
