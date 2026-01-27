+++
date = '2026-01-27T13:29:37-08:00'
title = 'Scripting Agents Without the Cliff'
featured_image = '/images/hero/cliffs-of-insanity.jpg'
+++

I tried [coding in English](/notes/coding-in-english/). It was fun attempting to build real software with nothing but natural language and the primitives of a CLI agent. I learned a lot, but I also hit some walls.

<!--more-->

Here's the thing: conversational scripting is actually great for getting started. You chat with Claude Code or Codex, get it to do something useful, and if you want to reuse it, you drop it in a markdown file. So easy.

But the moment you want real control—loops that don't require three paragraphs of prose, branching logic that isn't ambiguous, or composition that actually scales—**you just still want code.**

The problem is, making that jump today feels like scaling the [Cliffs of Insanity](https://www.youtube.com/watch?v=ClzaP8HN2wc). Modern agent frameworks drop you into a world of account registration, configuration files, API key management, model selection matrices, and server lifecycle management. What started as "just tell the computer what to do" becomes a yak shave.

I wanted something in between.

## Thinkwell

I've been building a TypeScript library called [thinkwell](https://npmjs.com/package/thinkwell) that aims to preserve the simplicity of markdown scripting while giving you the power of a real programming language. The idea is simple: keep the parts that work (telling an LLM what to do in plain English) and replace the parts that don't (control flow, composition, deterministic logic) with TypeScript.

Here's a hello world:

```typescript
import { Agent } from "thinkwell";
import { CLAUDE_CODE } from "thinkwell/connectors";
import { GreetingSchema } from "./greeting.schemas.js";

/**
 * A friendly greeting.
 * @JSONSchema
 */
interface Greeting {
  /** The greeting message */
  message: string;
}

const agent = await Agent.connect(CLAUDE_CODE);

const greeting: Greeting = await agent
  .think(GreetingSchema)
  .text(`
    Use the current_time tool to get the current time, and create a
    friendly greeting message appropriate for that time of day.
  `)

  .tool(
    "current_time",
    "Produces the current date, time, and time zone.",
    async () => ({
      timeZone: Intl.DateTimeFormat().resolvedOptions().timeZone,
      time: new Date().toLocaleTimeString(),
      date: new Date().toLocaleDateString(),
    })
  )

  .run();
```

Notice what's missing: no config files, model selection, or temperature tuning. You connect to an existing CLI agent, describe what you want in English, define tools as inline TypeScript callbacks, and get back a typed result.

## The Fluent API

Thinkwell uses a fluent builder pattern that lets you chain together prompts, quoted content, and tools:

```typescript
const analysis = await agent
  .think(DocumentAnalysisSchema)  // expected output type
  .text(`Analyze this customer feedback...`)
  .quote(feedbackText, "feedback")  // include content
  .tool("analyze_sentiment", "...", schema, callback)  // add tools
  .run();
```

Each piece does one thing:
- `.think(schema)` sets up the expected output type
- `.text()` adds prompt text
- `.quote()` and `.code()` include content blocks
- `.tool()` installs MCP tools with TypeScript callbacks
- `.run()` executes and returns a typed result

The tools are real MCP tools that the agent can call, but you define them as simple async functions. No server setup, no JSON-RPC boilerplate.

## A Real Example: Unminifying JavaScript

To show a more realistic example of how agent scripting can be both easy and powerful, I built a JavaScript unminifier (in under 300 lines!). The pipeline:

1. Pretty-print with Prettier (deterministic)
2. Convert UMD wrapper to ESM (LLM)
3. Extract list of minified functions (LLM)
4. Analyze each function to suggest better names (LLM, parallelized)
5. Apply the renames (LLM)

Here's the parallel analysis step:

```typescript
const limit = pLimit(5);
const batches = _.chunk(functionList.functions, 30);

const results = await Promise.all(
  batches.map((batch) =>
    limit(async () => {
      return agent
        .think(FunctionAnalysisBatchSchema)
        .text(`Analyze each function and suggest better names...`)
        .code(esmCode, "javascript")
        .run();
    })
  )
);
```

It's just standard JavaScript promises with a [popular concurrency limiter](https://www.npmjs.com/package/p-limit), calling out to the LLM for the fuzzy parts. The deterministic parts (batching, rate limiting, aggregating results) are just ordinary TypeScript.

## What Makes This Different

The key insight is that thinkwell doesn't try to replace your CLI agent—it wraps it. You're still using Claude Code, or any agent that speaks [ACP](https://agentclientprotocol.com/overview/introduction). The library just gives you a better way to compose prompts, manage tools, and handle responses.

## The Spectrum

I've come to think of agentic automation as a spectrum:

```goat {.f7}
                     Control
                        ^
                        |                    .-------* Frameworks
                        |                   /
                        |                  /
                        |                 |   <-- the cliff
                        |        *--------'
                        |     Thinkwell
                        |    /
                        |   * Markdown
                        |  /
                        | * Chat
                        +-------------------------> Setup
                       None                      High
```

Thinkwell sits in a sweet spot: more control than markdown, less ceremony than a full framework. It's for those times when you've outgrown "just tell it what to do" but aren't ready to commit to a complex SDK.

## Try It Out

Thinkwell is still early, but it's on [npm](https://npmjs.com/package/thinkwell) and [GitHub](https://github.com/dherman/thinkwell) if you want to experiment. I'd love feedback on the API design and what kinds of workflows people want to build.

And if you're still using pure markdown scripts? That's fine too. Sometimes the simplest tool is the right one. But when you need more, it's nice to have a path that doesn't involve scaling a cliff.

###### image credits: [John Finkelstein](https://unsplash.com/photos/cliff-near-sea-ci9HzWTIVas)
