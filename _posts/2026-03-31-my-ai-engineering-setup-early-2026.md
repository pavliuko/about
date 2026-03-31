---
layout: post
title: "My AI Engineering Setup, Early 2026 (Co-Authored-By: Me)"
---

I'm not much of a writer, so I did what I do best. Set up the instructions, pointed the agents at the task, and reviewed what came back. This article was written using the same setup it describes.

So, quick intro. I'm an iOS engineer. I build apps, I ship features, I maintain projects mostly on my own. I've been deep into building with AI for a few years now, and the setup has gotten pretty involved.

When I was a kid, I loved "nonsense generator" websites. They produced text that looked perfectly coherent at first glance, proper sentences, real-looking grammar, but when you actually read it, there was zero meaning. Back then, that blew my mind. I think my interest in AI started right there, haha.

## The Journey: Copilot to Cursor to CLI

I got into AI engineering like a lot of us, starting with Copilot and ChatGPT. I've been keeping up with all the news, and luckily I've had the chance to try out everything new as it drops. Still do. So my approach keeps evolving right alongside the market.

The Cursor phase was genuinely fun. Smart completion, inline edits, and sometimes it just feels easier to write code by hand with a good autocomplete whispering in your ear. I still switch to Cursor for that feeling. But here's the thing about being an iOS engineer: you can't escape Xcode. So you end up living in two editors whether you like it or not. I was constantly jumping between them, and just switching apps wasn't enough. I needed the right file, the right line, the right context on both sides. So I built [cursor-behavior-for-xcode](https://github.com/pavliuko/cursor-behavior-for-xcode) and found [vscode-xopen](https://github.com/moderato-app/vscode-xopen), and now I have a loop where I'm switching IDEs effortlessly.

When Karpathy introduced "vibe coding," I jokingly said I invented "vibe searching." At that time, I maintained a large project that forked firefox-ios, with a ton of functionality piled on top. The codebase was huge. I had no idea where things were or what they were called.

> Dropped into a giant repo. Forked, bloated, unfamiliar. No idea where things are or what they're called. I just tell AI what I need, it does the heavy lifting. This is what I mean by vibe searching.
>
> — [@pasashque](https://x.com/pasashque/status/1911743807206326779)

That experience kept coming back to me. Different ways of asking AI agents are useful well beyond writing code. They help you understand unfamiliar codebases and pick up new knowledge in the process.

Eventually I moved to a terminal-based agentic workflow. The shift felt natural. When most code comes from agents, the chat window matters more than the editor. I still use Cursor for hands-on editing on a regular basis, but the center of gravity shifted.

I'm also using Wispr Flow for voice input. Talking to your terminal sounds weird until you try it with a CLI-first workflow. Then it just clicks.

## What My Day Actually Looks Like

The daily rhythm goes like this: explore the problem, point the agent in the right direction, review what comes back, iterate. Less typing, more thinking.

PRs, issues, PRDs, other docs, even commit messages. The AI agent works as my personal assistant taking all the tedious work, giving me the opportunity to do something more fun. And the shift is real. I write way less code by hand now. Most of my day is directing and reviewing.

## The Instruction Architecture

Alright, this is the core of the whole thing. If you take one idea from this article, let it be this:

You don't prompt. You build systems that eliminate prompting.

My actual in-session prompts are almost nothing. A short phrase at most. All the heavy lifting lives in skill files, agent rules, AGENTS.md files, and doc references. I traded upfront documentation effort for near-zero per-task prompting cost.

The technique: write the prompt once as infrastructure, then invoke it by name forever.

Let me break down how this actually works.

### The Layered System

I've built a multi-tier instruction architecture.

At the top sits AGENTS.md, the root config that the agent loads every session. Project-wide rules live here: git safety, formatting conventions, architecture constraints. It also references agent rules, which are always loaded too. Then skills and commands that only get pulled in when they're actually relevant.

I'm trying to keep things agent-agnostic, so I maintain a shared AGENTS.md and wire agent-specific files like CLAUDE.md back to it. Do I actually use a bunch of different agents day-to-day? Unfortunately no. Cursor-cli and Codex sometimes, OpenCode and Pi is just wishful thinking at this point. But the structure is there.

### Skills: The Sweet Spot

Skills are great. They fill the spot right between rules and commands, allowing the AI agent to decide when to load context on its own. They're not dumped into every conversation, just the relevant ones. Skills can also be followed with templates and scripts, which is a pretty promising direction.

Now, honesty time. Discoverability still doesn't perfectly work for me. The agent doesn't always pick up the right skill at the right time. While I'm still looking for a solution, I keep dropping the skill name into the prompt whenever I don't want to roll the dice.

### Commands as Workflow Automation

Commands can be short or super detailed. Some are aimed at interviewing me first and then launching an agent. Others just fire off skills directly.

One of my favorite: `/new-ui-feature`. It asks about designs, functionality descriptions, and examples to follow. It asks "Are there any existing screens in the app I should follow as a reference?" Then it loads those references before writing a single line of new code. It performs a large amount of routine work, which is incredibly convenient.

The pattern is always the same. Write the prompt once as infrastructure, invoke it by name forever.

### Docs and Agent Rules

I divide documents into two types. Agent rules are only for the agent. General documents are for both agents and humans to read. Agent-only rules can be prescriptive and verbose because no human needs to maintain them as documentation. They're instructions, not docs.

I keep AGENTS.md, skills, and commands concise and move detailed instructions into docs. The nice part is that skills and commands handle on-demand context well, loading external docs only when triggered.

### Sub-agents

I changed my mind on these. Dismissed them earlier, recently accepted them again.

The main reason for using them is simple: they combat context creep. When a conversation gets long, quality drifts. Spinning up a fresh sub-agent with focused instructions fixes that.

## How I Actually Talk to the Agent

### The Three Request Patterns

I looked at how I actually interact with the agent, and it breaks down into three patterns.

**Explore first, then act (~50% of requests).** This is the big one. My most common pattern is asking an open-ended question before doing anything. Let the agent investigate, then decide direction based on findings. I almost never jump straight to "build this." It's always reconnaissance first, implementation second.

**Reference-based prompting (~25%).** I build one thing right, then let the agent use it as a blueprint. Every similar task after that starts from a working reference instead of a blank spec. And here's the part I really like: this creates space for personal growth in the domain. As your understanding improves, you build better references, and the agent produces better results. The quality goes up with you.

**Behavioral specs, not code specs (~20%).** I describe what the user should experience, not how to build it. The agent translates user flows into implementation. When I want to think at the implementation level, I drop into the code myself.

What I almost never do: step-by-step implementation instructions, long detailed specs upfront, or pasting code and asking to modify it.

## The Support Stack

### Agent Plus CLI

Agent plus CLI. That's the combo. Not an IDE-first workflow. I build simple command-line tools from time to time, combining them with dedicated custom skills. The terminal became the center of everything.

### XcodeBuildMCP

[XcodeBuildMCP](https://www.xcodebuildmcp.com/) is my most-used MCP server. I'm not a big fan of MCPs in general, but this one earns its spot. I use it for building, running the compiler, and tests. I don't use it beyond that yet, but I'm exploring iteratively. Recently started looking into UI automation, which could open up a lot of possibilities.

### Sosumi (Apple Docs MCP)

I use [Sosumi](https://sosumi.ai/) for Apple documentation search, and it works well. The agent can look up framework APIs on the fly, which means it's not guessing at signatures or inventing deprecated patterns. As a bonus, it helps me design better APIs for my own code by pulling ideas straight from Apple's documentation. When you're building on top of UIKit or SwiftUI, having that reference available in the conversation makes a noticeable difference.

### PR Review Pipeline

I'm pretty much the only contributor on the projects I work on. No teammates, no supervisor. So AI code review isn't a nice-to-have, it's just how review happens.

My CI pipelines have different workflows that check the code, plus instructions for AI agents. I also built [swift-concurrency-reviewer](https://github.com/pavliuko/swift-concurrency-reviewer), a plugin that reviews PRs specifically for Swift concurrency issues. It's built on top of Antoine van der Lee's [Swift Concurrency Agent Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill), which handles the actual analysis with decision trees and patterns. The plugin just orchestrates the review and formats the output.

All of this catches things I'd miss reviewing my own code. Not a replacement for a human colleague, but a solid second pair of eyes when there's no one else around.

### Testing

Agents are already writing more code than humans, and the only way to keep that safe and reliable is to wrap it in strict engineering practices. Tests are maybe the most important part of the self-verification loop I've built into my projects. Agents are forced to keep tests green, and they learn from them along the way. TDD feels like a natural next step here, and I'm in the process of adopting it.

Manual and semi-manual testing have also turned out to be surprisingly valuable. Sometimes you just need to tap through the app yourself, or even better, have someone else do it. Same as it was before the AI era.

## Problems, Tradeoffs, and the Complexity Tax

### The Review Problem

I think the hot topics for 2026 are gonna be around how engineers collaborate in this new reality. Starting with the basics like "how do we even review code at the same pace now?" and scaling up to building teams and processes.

When it comes to team management, I honestly don't have much to say at this point. But on the review side? The gap is real. The tools help, they really do. But they don't replace the kind of nuanced review a human colleague gives. For a solo contributor, you feel that gap every day.

### The Complexity Growth

In addition to building products, I now also build systems for AI tools. The instruction architecture, the skills, the rules, the review pipelines. All of it takes time. Real time. And it keeps growing.

As the setup grows, staying agent-agnostic gets harder and harder. I'm still trying, but in practice most of my time goes into Claude Code. Every new capability means new rules, new edge cases, new maintenance. The time investment is real and ongoing.

## The Agent Is You

This is how I see working with agents. The agent is a direct continuation of yourself. Your knowledge, your taste, your way of getting things done.

Everything you invest into the setup is just you, encoded. The agent doesn't replace what you know. It scales it. And as you grow, the system grows with you.

That's also why I think every contributor on a project should have their own collection of skills. Shared project rules make sense. But skills are personal. They reflect how you think, how you approach problems, what shortcuts you've built up over years. Two engineers on the same repo will work differently, and their agents should too.

## It's Just Fun

It brings a lot of fun to day-to-day routine, and this is simply awesome.

That kid who was amazed by nonsense generator websites? He grew up to build systems where AI agents ship production code while he directs traffic from the command line. Makes sense, kind of :).
