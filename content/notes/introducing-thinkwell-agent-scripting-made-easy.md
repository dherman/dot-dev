+++
date = '2026-02-12T11:47:32-05:00'
title = 'Introducing Thinkwell: Agent Scripting Made Easy'
featured_image = '/images/hero/typewriter.jpg'
+++

Last year's explosion of agentic concepts was dizzying: MCP, AGENTS.md, skills, subagents, hooks, plugins, specs… we're all running so fast to keep up, it's easy to miss that we're on someone else's treadmill: the coding agent.

<!--more-->

I think 2026 is the year more people start to realize we don't have to keep waiting around for the next feature drop from the big platforms. It turns out [you can write an agent](https://fly.io/blog/everyone-write-an-agent/). Not theoretically. _You can actually do it._ No really, you can! Your own agent, for your own workflow, doing exactly what you need.

The problem is that writing an agent today kind of sucks.

## The Framework Gap

The agent frameworks that emerged in 2022 and 2023 were built for a different era. They're [too low level](https://blog.bryanl.dev/posts/agent-framework-vision/): they immediately drop you into managing details like model selection, API key management, temperature tuning, provider-specific quirks, and server lifecycle boilerplate.

On the other end of the spectrum, people try to script their entire agent workflow in markdown and feed it to a coding agent. I've [tried this myself](/notes/coding-in-english/) and it sort of works. But as soon as you need a loop, a conditional, or any kind of composition, you start finding yourself writing paragraphs of INCREASINGLY SHOUTY PROSE to express what would be one line of code.

## Thinkwell

Over the last few weeks, I've been building a TypeScript library called [Thinkwell](https://thinkwell.sh) to fill this gap. It was born out of running into one engineer after another who wanted to build their own agents but kept bouncing off the tooling. They didn't want to learn an elaborate SDK, they just wanted to hack.

And the use cases weren't exotic, they were the kind of thing any software team runs into:

- Porting code from one language to another
- Upgrading libraries with backwards-incompatible API changes
- Cleaning up a sequence of git commits into a coherent narrative

These are tasks where you want an LLM for the fuzzy reasoning parts but real code for the control flow, concurrency, and deterministic logic. Thinkwell lets you combine both.

I've done a [bunch of tinkering](https://github.com/dherman/thinkwell/pulls?q=is%3Apr+is%3Aclosed) since I first posted about the [developer experience](/notes/scripting-agents-without-the-cliff/) I'm going for. Two weeks and 40k lines later, I'm pretty happy with [where it's landed](https://npmjs.com/package/thinkwell).

### Just `open()`

Connecting to an agent is a single function call:

```typescript
import { open } from "thinkwell";

const agent = await open('claude');
```

You pick from a set of well-known CLI agents by name—`'claude'`, `'codex'`, `'opencode'`, `'kiro'`, `'gemini'`—and the library handles the rest. Any agent that speaks [ACP](https://agentclientprotocol.com/overview/introduction) just works.[^1]

### Everything Starts with `.think()`

Every interaction with an agent follows a simple, fluent API:

```typescript
/**
 * A friendly greeting.
 * @JSONSchema
 */
interface Greeting {
  /** The greeting message */
  message: string;
}

const greeting: Greeting = await agent
  .think(Greeting.Schema)
  .text("Create a friendly greeting for the user.")
  .run();
```

You define your expected output as a TypeScript interface, describe what you want in plain English, and get back a typed result. The `@JSONSchema` annotation generates your schema automatically—no manual JSON Schema crafting.

### Subagents via Promises

Need to split work across independent subagents? Every agent session maintains its own conversation context. You can kick off as many as you need and orchestrate them with ordinary TypeScript:

```typescript
import pLimit from "p-limit";

const limit = pLimit(5);

const results = await Promise.all(
  files.map(file =>
    limit(async () => {
      const session = await agent.createSession();
      const analysis = await session
        .think(Analysis.Schema)
        .text(`Analyze this file for security issues.`)
        .code(file.contents, "typescript")
        .run();
      const fixes = await session
        .think(FixSuggestions.Schema)
        .text(`Suggest fixes for the issues you found.`)
        .run();
      return fixes;
    })
  )
);
```

Each session remembers its conversation history, so the second `.think()` can reference the first without re-sending context. And concurrency control is just promise libraries and `async`/`await`.

### Zero-Config CLI

If you're not familiar with Node or TypeScript, you can let your coding agent help you write the code, and you don't need to install anything other than the Thinkwell CLI:

```bash
brew install dherman/thinkwell/thinkwell
```

Then just run your script:

```bash
thinkwell myscript.ts
```

This lets you get started without even learning the Node basics or dealing with any config files. You just write a `.ts` file and run it. The CLI handles TypeScript compilation, schema generation, and dependency resolution behind the scenes.

### One-Touch Setup

Is your script working out and you want to maintain it longer-term? Use `thinkwell new` or `thinkwell init` to initialize a proper project with the right dependencies and build steps. My philosophy is "pay as you go": start quick, grow when you're ready.

### Documentation

Thinkwell now has a proper home at [thinkwell.sh](https://thinkwell.sh), with quick start instructions and full API docs.

## Why This Matters

There's an explosion of things people want to automate with LLMs, and the barrier to entry is still too high. You shouldn't need to understand provider APIs, manage credentials, or configure build toolchains just to script an agent. You should be able to go from idea to working automation in minutes.

That's what I'm building towards. [Come play!](https://thinkwell.sh)

###### image credits: [Luca Onniboni](https://unsplash.com/photos/teal-and-black-typewriter-machine-4v9Kk01mEbY)

[^1]: Or if it doesn't, it's a bug and you should [tell me](https://github.com/dherman/thinkwell/issues)—I'll fix it. Better yet, come join me and we'll fix it together. I'm always looking for contributors!
