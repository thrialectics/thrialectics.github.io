---
title: "Context Engineering with Threadlinking: Like Git for Decision History"
date: 2026-02-09
---

Dear Friends,

If we're twitter mutuals, you might have seen me post about a context engineering tool I recently built for Claude Code called Threadlinking. A few people have asked for a writeup, so here it is.

The idea predates Claude Code. In spring 2025, I was generating files through ChatGPT conversations that would end up in my Downloads folder with no record of why they existed. So I built a prototype: a CLI tool with a browser extension that captured conversation URLs and linked them to files under a named "thread," a project-level tag connecting files to their context.

Months later, once I was deep into Claude Code, I realized there was a simpler implementation if I made Threadlinking "Claude Code native." The same problem had followed me; I'd build something useful in a session, lose all the reasoning when the conversation ended, and have to re-explain everything next time. Half the time I'd forgotten the details myself.

So I updated Threadlinking to be a tool for Claude as much as a tool for me.

**[Threadlinking](https://github.com/thrialectics/threadlinking)** is an open-source context preservation tool for Claude Code. It saves the reasoning, snippets, and summaries from your conversations alongside the files they produced, so that when you come back later, the origin story is still there.

---

## Code History vs. Decision History

We already have tools for tracking *what* changed in our code. Git gives you a complete record of every line added, removed, modified, and when. Which is great, but the limitation is that git doesn't tell you the "why" behind your changes. The commit message might say "switch to PostgreSQL," but it won't tell you that you considered MongoDB first, rejected it because your payment system needs ACID transactions, and almost went with SQLite before realizing you'd need concurrent writes.

That reasoning is a different kind of artifact than the code itself. I've started calling this **decision history**, and the more I worked with Claude Code, the more I noticed my decision history was invisible in most of my projects. Not just for Claude's memory, but mine, too.

This isn't an idiosyncratic problem. Anyone building with AI coding agents or other assistants across multiple sessions runs into it. Common workarounds are `CLAUDE.md` files, manual notes, or pasting summaries into the start of each new conversation. These help, but they're static. You have to maintain them by hand, and they don't capture context as it's being generated.

The insight that shaped Threadlinking was that the unit of context preservation should be the **project**, not the conversation. A conversation starts and ends and gets lost. But a project lives across many sessions, often across months. What I actually cared about preserving wasn't "what happened in this chat" but "why does this project look the way it does right now."

---

## Threads, Snippets, and File Links

The core concepts in Threadlinking are pretty simple.

A **thread** is a named container for context. You create one per project or idea (not per task, not per session). I have threads called `personal-website`, `threadlinking`, `byemarianne`. They accumulate context over time as I keep working on these projects across many sessions.

A **snippet** is a short piece of context that gets saved to a thread. It's usually a sentence or two explaining a decision, a trade-off, or some reasoning that would otherwise disappear with the conversation. "Chose Eleventy over Next.js because the goal was zero client-side JavaScript." That kind of thing.

**File links** connect specific files to threads. When you link a file, you're creating a durable connection between "this code exists" and "here's why." Later, when you or Claude run `threadlinking explain path/to/file`, you get back every snippet from every thread that file belongs to. The full origin story.

The naming convention matters. Good thread names are project-level: `personal-website`, `auth-system`, `client-acme`. Bad thread names are task-level: `fix-bug-123`, `refactor-tuesday`. Threads are meant to be long-lived. They grow over time, session after session.

---

## How It Actually Works

Threadlinking plugs into Claude Code through three components. I'll explain each briefly, and if you're not already familiar with MCP, I'll give you the short version: **MCP** (Model Context Protocol) is the open standard that lets AI assistants use external tools. Anthropic created it, and in December 2025 they donated it to the Linux Foundation, where it's now co-governed by OpenAI, Google, Microsoft, and others. It's how Claude Code talks to the outside world.

#### The Hook

When you install Threadlinking, it registers a hook that fires every time Claude creates or edits a file. The hook quietly records which files were touched during your session. It doesn't capture any content or reasoning on its own. It just notes "this file was modified" and adds it to a pending list.

What this means in practice is that at the start of your next session, Threadlinking can tell you: "These files were edited but aren't linked to any thread yet." It's a nudge. You can link them or ignore them. Files that stay unlinked for 30 days get cleared from the pending list automatically.

#### The MCP Server

The MCP server gives Claude direct access to the threadlinking system. Claude can create threads, add snippets, link files, search across your context, and explain the history behind a file, all without you having to type CLI commands.

This is what I meant earlier about making it a tool for Claude as much as for me. Claude can proactively save context when it notices significant decisions being made. When you start a new session, Claude checks for relevant threads and gets oriented on its own. I've found this changes the experience of returning to a project quite a bit, because Claude doesn't just have the code in front of it; it has the story of the code.

#### The CLAUDE.md Instructions

The third piece is a set of instructions that get appended to your `CLAUDE.md` file during setup. These teach Claude what's worth saving and what isn't. Architectural decisions and trade-off reasoning: yes. Typo fixes and dependency updates: no. This matters because if everything gets saved, the context becomes noise. The instructions are basically judgment-encoding for what constitutes meaningful context.

---

## Local-First, No Cloud

All of this lives in a single JSON file at `~/.threadlinking/thread_index.json`. No cloud database, no account to create, no telemetry. Your context stays on your machine.

I want to be straightforward about the privacy picture, though. If you're using Threadlinking through the MCP server, Claude reads your context in order to work with it, which means it passes through Anthropic's infrastructure like anything else in your conversation. Threadlinking doesn't add any *new* data exposure beyond what you're already sharing by using Claude Code, but it doesn't eliminate it either.

For people who are more privacy-conscious about this, Threadlinking also works as a standalone CLI tool. You can use it entirely by hand, without the MCP server or hooks, as a kind of "git for decisions." You save snippets, link files, and search your context yourself. Nothing touches an LLM unless you choose to set up the MCP integration.

Under the hood, the file operations use atomic writes and locking to prevent corruption when the CLI and MCP server are both trying to read and write at the same time (which happens more than you'd think). If the JSON file somehow gets corrupted, the storage layer backs it up automatically and falls back to a clean state.

---

## Semantic Search

Beyond basic keyword matching, Threadlinking also supports **semantic search**. This is worth explaining because it's one of the more useful features for cross-project discovery.

The semantic search runs a local embedding model (`all-MiniLM-L6-v2`) that converts your snippets into 384-dimensional vectors and stores them in a local vector database called Vectra. What this means in practice is that you can search by meaning rather than exact words. If you search "why did we choose this database," it will find a snippet about "went with PostgreSQL over MongoDB for relational integrity" even though those words don't overlap at all.

The embedding model downloads once (about 30MB, cached locally) and runs entirely on your machine after that. No API calls, no tokens consumed, no data sent anywhere. To set it up, you run `threadlinking reindex`, which builds the vector index from your existing snippets. After that, new snippets get indexed automatically as you add them.

---

## What It Looks Like

I want to give you a sense of what using Threadlinking actually feels like, so here's what happens in my own workflow.

When I start a session, I'll ask Claude to run `threadlinking list` to see where things stand. Here's what mine looks like right now, with 18 threads across my projects:

![Threadlinking list output in Claude Code, showing threads with snippet counts, file counts, and descriptions in a table view](/images/screenshot2.jpg)

Each row is a project I've been working on. Below the thread table, there's a list of files that were edited in recent sessions but haven't been linked to a thread yet. From here Claude might ask if I want to attach them, or I'll tell it to.

During a session, when Claude and I are making decisions, Claude saves the reasoning as snippets. For this website, for example, one of the snippets reads:

```
threadlinking snippet personal-website "Chose Eleventy over Next.js.
The goal is a minimalist site with no client-side JavaScript. Eleventy
generates static HTML, which aligns with the performance and simplicity
requirements."
```

I didn't type that. Claude did it on its own because the CLAUDE.md instructions taught it to recognize that kind of decision as worth saving.

Here's a real example from a different session, where Claude saved a snippet about merging the search commands in Threadlinking's own codebase:

![Claude saving a snippet via the MCP server, showing the full tool call with content and tags](/images/screenshot1.jpg)

When I come back to a project weeks later and wonder why I made a particular choice, I (or Claude) can run:

```
threadlinking explain .eleventy.js
```

And I get back the full context. When the decision was made, what the reasoning was. The origin story. Here's the tail end of what `explain` returns for Threadlinking's own MCP server file, showing recent architecture decisions:

![Threadlinking explain output showing snippets 20-22, including MCP config decisions and git-for-decisions exploration](/images/screenshot6.png)

If I can't remember which of my projects had a conversation about, say, cryptographic key generation, I can use semantic search:

```
threadlinking semantic-search "cryptographic key generation"
```

This searches across all my threads by meaning and returns the relevant ones ranked by similarity, surfacing connections across projects you might not have thought to look for:

![Semantic search results showing matches across threadlinking-planning, serpent-dove, project-search, and intention-primitive-language threads](/images/screenshot7.png)

---

## The Bigger Picture

I'm an ARG designer and the founder of [Klew Studio](https://klewstudio.com), where I work on narrative intelligence. I came to software through AI. Andrej Karpathy called this phenomenon "vibe coding" last year, and he recently proposed a follow-up term, **"agentic engineering,"** for the more rigorous version of the practice. I like the new term better.

I don't have a traditional engineering background. I'm a designer and strategist who started building software because AI tools made it possible. Context preservation matters to me *because* of this. When I'm working with Claude on a codebase, I need the reasoning to persist between sessions. I can't always reconstruct it from the code alone. But I think this problem exists regardless of skill level. Even experienced engineers lose track of *why* when the conversation that produced *what* disappears.

There's a connection to how I think about narrative design that I want to draw out. In ARGs, one of the central problems is maintaining story continuity across many touchpoints, participants, and timeframes. The story has to cohere even when people encounter pieces out of order. Threadlinking is that same problem applied to code: how do you maintain the coherence of a project's story across sessions, contributors, and time?

A codebase tells a story. Git is the record of what happened. Threadlinking is the record of why.

---

## Getting Started

Threadlinking is [open source on GitHub](https://github.com/thrialectics/threadlinking) and published on npm. If you want to try it:

```
npx threadlinking init
```

The `init` command walks you through the full setup. It configures the hooks, the MCP server, and the CLAUDE.md instructions. The whole process takes about a minute.

![Threadlinking init setup process, showing hook installation and CLAUDE.md configuration](/images/screenshot3.jpg)

I'm currently working on getting it listed in the MCP server directories (Smithery, Glama, mcp.so) and exploring IDE extensions for VS Code. The [roadmap](https://github.com/thrialectics/threadlinking/blob/main/ROADMAP.md) is public if you're interested in where it's headed or want to contribute.

I'd really like to hear how this resonates with your own experience. Have you built systems for preserving context between sessions? Have you felt the frustration of losing it? Let me know via comments or [DM me on X](https://x.com/thrialectics).

So Long,

Marianne 🩵
