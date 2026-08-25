+++
date = '2026-08-24T14:34:05-07:00'
title = 'The Monorepo Spectrum'
featured_image = '/images/hero/spectrum.jpg'
+++

I'm not going to try to convince anyone today of whether monorepos are a good or bad idea. Instead, I want to focus on the underappreciated and fundamental fact that **a monorepo is not a monorepo is not a monorepo.**

<!--more-->

How to structure and organize code is a surprisingly subtle topic, with big implications for how software gets built.
The main tension is between two important engineering constraints: **independence** and **coordination.**
Independence enables organizations to parallelize, which is key to scaling most work. But coordination use cases, while typically less common, are no less important, and cover a range of needs: atomic releases, cross-cutting product changes, coding standards, API refactorings.

The repository comes into play because it presents an atomic view of the state of a codebase. Monorepo solutions take advantage of this atomicity to optimize coordination; multi-repo solutions instead inherently optimize for independence. Whichever approach is taken, that choice leaves a gap that has to be filled with additional tooling and engineering practices.

There are real, practical implications in the trade-offs between the two. But what I find most unsatisfying about most debates is what almost always gets left out: **what scale are we talking about?** Trying to draw conclusions about how to organize a weekend hobby app based on how a bigco runs its entire software engineering organization is essentially a category error.

## A Simple Framework: Team Size

My usual advice to anyone considering a monorepo is to start by trying to project the scale over the next several years. In my experience, the most determinative question is the number of concurrent developers, which is easier to estimate in units of [two-pizza teams](https://martinfowler.com/bliki/TwoPizzaTeam.html) on a logscale, leading to simple t-shirt sizes:

| Size | Scale       | Examples                                        |
|------|-------------|-------------------------------------------------|
| S    | 1 team      | Small app; collection of related services       |
| M    | 10 teams    | Mature product                                  |
| L    | 100 teams   | Product suite; platform monorepo; small company |
| XL   | 1000+ teams | Large company                                   |

This is only the first of many questions. But it's useful for:

**Dismissing sloppy arguments.** Atomic commits are [not "refactoring for free"](https://abseil.io/resources/swe-book/html/ch22.html#barriers_to_atomic_changes) at any scale. You may not need Bazel or Buck for a small repo. And be skeptical of vendors making claims about tools built for small or medium sized monorepos by reference to the reputations of megacorporations with totally distinct infrastructure.

**Predicting infrastructure needs.** Not every piece of infrastructure is needed on day one. By thinking about the growth trajectory of the codebase, teams can make developer tooling investments by need. Keep in mind, though, that these predictions may shift as the physics of software is changing.[^1]

**Focusing debate.** Scale is a useful tool for helping ensure that people are having the same conversation.

That's my "monorepo spectrum" model: easy enough to remember but explanatory enough to bring structure to a complex and overwhelming topic that's all-too-prone to hand-waving and dogma.

[^1]: For example, I have found that the need for merge queues (aka merge trains) arises much sooner now that engineers and agents are generating a higher volume of PRs.

###### image credits: [Kevin O'Connor](https://unsplash.com/photos/woman-walking-in-hallway-JxSLigoB-6A)
