+++
date = '2026-08-26T12:30:20-07:00'
title = 'What Is a Monorepo?'
featured_image = '/images/hero/crayons.jpg'
+++

If monorepos are [different beasts at different scales]({{% relref "the-monorepo-spectrum.md" %}}),
do they have anything in common? Or does having a single term for such divergent technologies
do more harm than good?

<!--more-->

To be sure, some engineers[^1] complain about the squishiness of the word. One repo for what?
At what scope? Isn't every repo a monorepo? What value does this word even provide when we could just say "repo?"

But the [descriptivist](https://en.wikipedia.org/wiki/Linguistic_description#Descriptive_versus_prescriptive_linguistics)
angel on my shoulder reminded me that terminology persists in culture for a reason.[^2] So it's worth
unpacking why. The power of this abstraction is that it conveys something about the _intent_ of organizing a codebase
into a single repository. Let's attempt a working definition:

<!--
the ambiguity of
"monorepo." One repo for what? At what scope? In snarkier moments I've even complained, isn't every
repo a monorepo?

But my [descriptivist](https://en.wikipedia.org/wiki/Linguistic_description#Descriptive_versus_prescriptive_linguistics)
side won out when I realized that there's an _intent_ 

Well, I might be a pedantic engineer, but I'm also a [descriptivist](https://en.wikipedia.org/wiki/Linguistic_description#Descriptive_versus_prescriptive_linguistics). The word "monorepo" has stuck
around because clearly it's conveying something people find useful. In fact, 

If the term "monorepo" weren't
a useful term, it wouldn't be so ubiquitous. And in fact, there is a serviceable working
definition that captures what people generally mean with a bit more precision:
-->

> A _monorepo_ is a codebase with guaranteed atomicity for code merges and software release candidates.

<!--
In other words, if we were _really_ pedantic, we might complain that monorepo is just another word for "repo."
(Every repo is a single repo!) But this misses something about the intent: a monorepo is an organizational
strategy that uses a single repo for a codebase that could otherwise be factored into multiple repos:
-->

In other words, if we take as fixed that we have some codebase we're working with, a monorepo
is a specific strategy for how to organize that codebase for the purpose of particular workflows.
In particular, monorepos optimize for atomicity of code merges:

![polyrepo-monorepo](/images/polyrepo-monorepo.png)

Where a polyrepo assembles release candidates by selecting specific revisions of components in a
dependency graph, a monorepo subsumes the source of the dependency graph and guarantees that
all changes anywhere across the graph are atomically merged into a single-threaded history.

## Benefits of Monorepos

A comprehensive, single-threaded codebase has strong implications for productivity:

* Builders merge their work into a single place, simplifying the day-to-day process of upstreaming feature work.
* Cross-cutting, even incompatible changes to shared abstractions and their use sites (up to a certain scale...) can be made without complex, multi-stage merge-and-release orchestrations.
* Tech leads enjoy easier oversight of the codebase (e.g., coding standards, external dependency curation).
* Searching, navigating, and understanding code relationships across the codebase is internally consistent since builders operate on coherent snapshots.
* Internal dependencies cannot, by definition, fall out of date.
* Release candidates can be fully assembled as part of the build without requiring additional machinery.

## Challenges of Monorepos

Monorepos are not a magic wand. As with polyrepos, they come with their own challenges:

* Build scalability: Advanced tooling is required to ensure that typical workflows that change only small portions of the codebase require a minimum of rebuilding.
* Test hygiene: Flaky tests are far more destructive to productivity in a large repo: one team's unpredictable test can ruin test repeatability for everyone across the entire codebase.
* Storage and retrieval scalability: Local disk space usage, repo sync speed, rate limiting, efficient and safe merging can become bottlenecks as repo size increases.
* Governance: Naive ownership systems that configure roles (e.g. reviewer privileges) and responsibilities (e.g. on-call) on a per-repo boundary do not scale; monorepos require sub-repo governance solutions.
* Privacy: Even in inner source settings, some organizations require restricting visibility to certain sensitive parts of a codebase. This may be supported by some off-the-shelf SCM systems but certainly not all.
* Validation: Even though _merges_ are atomic, a monorepo may or may not guarantee that _validation_ of merges is performed atomically. Without solutions like merge queues or merge trains, which provide stronger guarantees about the atomicity of built+test+merge, race conditions can cause concurrently validated merges to break the tree.
* Independent release: Monorepos optimize for dealing with all components at the latest version. Supporting complex release orchestration semantics requires additional machinery, and some use cases like deploying emergency patches to older versions can be an awkward fit.

## Takeaway

This is not a complete list, but it's a place to start. My intent isn't to overwhelm but, short of a grand unified theory,
I feel at least I can ground a controversial topic with some structure.

Maybe some day we will get to a point where repo strategy is a straightforward and automatable decision, but today it
still requires judgment and planning. And the costs and benefits are undoubtedly going to keep changing as the current upheaval
of software engineering continues. So for now, it's still a good idea for all engineers to learn these basics of repo design and their practical implications.

[^1]: Yes, I've been one of them.
[^2]: If you want to hear me go on a descriptivist rant, ask me about my impassioned defense of the word "transpiler."
