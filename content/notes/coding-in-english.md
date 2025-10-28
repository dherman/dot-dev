+++
date = '2025-10-28T12:32:41-07:00'
title = 'Is Coding in English Becoming Viable?'
featured_image = '/images/hero/ancient-script.jpg'
+++

Look, I'm starting from a position of skepticism.

<!--more-->

My take has been that natural language is a phenomenal tool for creativity and exploration, but poorly suited to precision or predictability. I've believed that intelligence will be in the tools of the future, but the artifacts we produce will still look more or less familiar as classic deterministic code.

But with the emergent phenomenon that AI agents are often "deterministic enough"—under the right conditions, even with all its creative randomness, an LLM can still end up achieving similar results when performing the same task—I've been checking my priors and exploring what coding in English might look like.

## The Experiment

Scripting in Markdown is already becoming ubiquitous. But now I'm curious: how far can we push it? Will we ever be able to build ambitious, large-scale software with something as fuzzy as natural language, or are scripts the most we can expect from English?

Over the past few days, I've been playing with composing agentic functionality using only the primitives available in modern CLI agents—no agentic SDKs, no APIs. I used Claude Code, although similar features exist in other tools. My constraints were simple:

- Work with the primitives given to me by the coding agent
- Pick a task that's too complex for a single LLM
- Write the logic in Markdown
- One exception: I allowed myself to add (via MCP) a few helpful but missing primitives.[^1]

The goal: build functionality that's
- **…reusable:** invokable from a user's coding agent
- **…determin-_ish_-tic:** repeatably produces results of similar quality
- **…modular:** composed from several components
- **…abstract:** minimal assumptions or overhead on the user's agent

### The Use Case

I built a reusable plugin called **[historian](https://github.com/dherman/claude-plugins/tree/main/plugins/historian)**, which takes a git feature branch and produces a fresh branch with the same net diff, but cleanly broken down into a logical commit sequence. Essentially, a tool to rewrite PR history into a proper narrative.

### The Architecture

The user invokes `/historian:narrate <PR description>`, which triggers a [tiny skill](https://github.com/dherman/claude-plugins/blob/main/plugins/historian/skills/rewriting-git-commits/SKILL.md?plain=1) that kicks off three specialized subagents:

1. **[Analyst](https://github.com/dherman/claude-plugins/blob/main/plugins/historian/agents/analyst.md?plain=1)**: Creates the master diff, develops a narrative, and plans the improved commit sequence
2. **[Narrator](https://github.com/dherman/claude-plugins/blob/main/plugins/historian/agents/narrator.md?plain=1)**: Loops over the commit sequence, instructing the scribe what to write at each step
3. **[Scribe](https://github.com/dherman/claude-plugins/blob/main/plugins/historian/agents/scribe.md?plain=1)**: Waits for instructions from the narrator and writes each individual commit

![fork-join](/images/fork-join.png)

### The Biggest Blocker

Imagine if you were told you can write as many functions as you want, but they can only be called by your program's `main()` function. Well, it turns out CLI agents like Claude Code [don't allow nested subagents](https://github.com/anthropics/claude-code/issues/4182). (I'm not sure why, but my guess is that they're worried about runaway inference costs.) This was by far my biggest frustration.

But I found I could work around some of this limitation by lauching all the necessary agents up front and letting them communicate. I built a little MCP server called [sidechat-mcp](https://github.com/dherman/claude-plugins/tree/main/plugins/historian/servers/sidechat-mcp) that lets agents pass messages to each other. For example, the **narrator**'s main loop hands a task to the **scribe** and blocks waiting for a result:

```markdown
For each commit in the plan, send a request to the scribe
via sidechat and wait for the result.

**For each commit**, use the `send_message` and
`receive_message` MCP tools:

**Example for commit #1:**

```typescript
// Send commit request to scribe
send_message({
  session: SESSION_ID,
  to: "scribe",
  message: {
    commit_num: 1,
    description: "Add user authentication models"
  }
})

// Log that we sent the request
log({
  session: SESSION_ID,
  agent: "NARRATOR",
  message: "Requested commit 1: Add user authentication models"
})

// Wait for scribe's response
receive_message({
  session: SESSION_ID,
  as: "narrator",
  timeout: 10800000  // 3 hours
})
```

## What I Learned

This is _starting_ to look like engineering. But there are still major limitations:

- Without transitivity, it's hard to see how abstraction scales. Maybe this will be a fundamental source of drag until inference costs drop low enough.
- When you need classic logic like loops and conditionals, writing it in English is just worse: verbose, distracting, and ambiguous.
- Although subjectively, my tests were converging on similar outcomes, coming up with objective criteria for evaluating nondeterministic software is still hard.
- Of course, using LLMs for all computation would be obscenely wasteful.

So is English the programming language of the future? In the purest sense, I'm still a no.

But the fact that LLMs can be so effective and even remarkably predictable tells me to keep an open mind. I'm excited to think we may be on the verge of exploring new models of not just coding tools but even what code itself could look like.

###### image credits: [Mark Rasmuson](https://unsplash.com/photos/photo-of-open-book-yri82tuk2TQ)

[^1]: I feel I kept to the spirit of the exercise, but feel free to check me on it! The [code is on GitHub](https://github.com/dherman/claude-plugins/blob/main/plugins/historian).
