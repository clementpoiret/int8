---
title: LLM-induced Skill Decay
# date: date '+%Y-%m-%dT%H:%M:%S%:z'
date: 2025-02-27T09:46:00+00:00
author: Clement POIRET
authorTwitter: clement_poiret1
cover: images/2_bloom.webp
tags: [programming, ai, personal development]
keywords: [cognition, programming, ai, skills]
description: Is AI creating lazy people? Certainly.
showFullContent: false
readingTime: true
hideComments: false
toc: true
tocBorder: true
---

A friend and I started programming in middle school using a cheap laptop my grandparents bought. Programming became a
way to tinker with the school's computers and create shitty, mostly useless software with Visual Studio and the .NET
Framework 3. Useless, yes, but I was having fun.

Years later, programming became my main source of income. When GitHub Copilot came out in closed beta, I
quickly applied to get early access. It gave me superpowers. I was able to get shit done asap, but work
became boring — I was losing the fun part: problem-solving.

Years later, programming became my primary source of income. When GitHub Copilot was released in closed beta, I quickly
applied for early access. It felt like I had superpowers. I was able to get things done ASAP, but work became boring.
I was losing the fun part: problem-solving.

I ended up disabling all code assistance and realized I was constantly waiting for the LLM to auto-complete me. I was
bored and lazy.

## What science says

A few days ago, I stumbled upon a paper from Microsoft: The Impact of Generative AI on Critical Thinking: Self-Reported
Reductions in Cognitive Effort and Confidence Effects From a Survey of Knowledge Workers[^paper].

The conclusion summarizes it well:

> While GenAI can improve worker efficiency, it can inhibit critical engagement with work and can potentially lead to
> long-term overreliance on the tool and diminished skill for independent problem-solving. Higher confidence in GenAI’s
> ability to perform a task is related to less critical thinking effort. When using GenAI tools, the effort invested in
> critical thinking shifts from information gathering to information verification; from problem-solving to AI response
> integration; and from task execution to task stewardship.

This isn't a new observation. Previous work from the same authors[^tankelevitch] already noted the shift toward an
increase in meta-cognitive processes — one's way to self-improvement through thinking, i.e., planning, monitoring, and
evaluating—decreasing one's reliance on existing knowledge and understanding.

## My two cents

This work is based on Bloom's Taxonomy (Fig. 1), a classification system of learning objectives organized as
hierarchical levels of cognitive complexity.

![Bloom's Taxonomy](/images/2_bloom.webp)

<center>Figure 1: Bloom's Taxonomy. It organizes learning objectives from the less cognitively demanding (remembering)
to the most demanding (creation). <a href="https://www.valamis.com/hub/blooms-taxonomy" target="_blank">Source.</a></center>

We started offloading parts of this taxonomy long ago. Books are a way to externalize long-term memory into an external
medium. Was that bad? No. Stephen Hawking wrote in his book *Brief Answers to the Big Questions* that books transcended
the biological limits of our DNA, storing ideas, discoveries, and culture.

I like the idea that delegating cognitive load to AI will free up brainpower to do things we couldn't mentally afford
to do, like spending more time creating. There is still debate on whether AI can truly be creative or if creativity is
the only thing we will be left with.

However, assuming humans are genuinely creative and not merely assembling pre-existing ideas, Bloom's Taxonomy
illustrates the dependence of creativity on foundational knowledge. If modern technological advances end up eroding the
base of the hierarchy, will we be able to create at all?

AI already exhibits clear benefits with the automation enabled by agents and LLMs, helping us solve our daily problems,
but we need to re-invent our symbiotic relationship, or we will live in an analogous situation to sedentarity, creating
obesity.

[^paper]: Lee, H.-P., et al., (2025). The Impact of Generative AI on Critical Thinking: Self-Reported Reductions
    in Cognitive Effort and Confidence Effects From a Survey of Knowledge Workers. CHI Conference on Human Factors in
    Computing Systems (CHI ’25). https://doi.org/10.1145/3706598.3713778

[^tankelevitch]: Tankelevitch, L., et al., (2024). The Metacognitive Demands and Opportunities of Generative AI.
    Proceedings of the CHI Conference on Human Factors in Computing Systems, 1–24.
    https://doi.org/10.1145/3613904.3642902
