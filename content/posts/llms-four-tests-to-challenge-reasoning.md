---
layout: default
title: "LLMs - Four Tests to Challenge Reasoning"
description: "Four diagnostic prompts that reveal how AI models handle contradictions, impossible geometry, temporal paradoxes, and infinite sets—tested across ChatGPT, Gemini, KIMI, Cerebras, and more."
date: 2026-02-12
slug: "llms-four-tests-to-challenge-reasoning"
author: "Ben Santora"
tags: ["LLMs", "AI testing", "reasoning", "logic puzzles", "spatial reasoning", "ChatGPT", "Gemini", "KIMI"]
---

*first published on dev.to - thanks to that community for their interest and contributions*

LLMs - Four Tests to Challenge Reasoning

Here are four challenges I've used that have revealed some very interesting patterns when given to the current crop of online LLMs. Consider them as diagnostic tools you can deploy that will prove to you that AI models do not all think in the same way.

These challenges are carefully designed to probe different methods of reasoning and target different types of weaknesses. I've tested them across multiple models, including ChatGPT, Gemini, Deepseek, KIMI, Qwen, Cerebras and others - the variations in responses between models really surprised me.

The rules are simple: Feed these to any AI system you're curious about. Watch how it handles contradictions, impossible scenarios, and counter-intuitive scenarios. Some models will get creative, some will get rigorous, others will spot the impossibility immediately. The first puzzle in particular really challenged the 'solver' models in interesting ways.

Challenge #1: "The Impossible Triangle"

The Setup: Three friends - Alex, Blake, and Casey - are standing in a triangle formation. Each person is exactly 10 feet from the other two. Alex says: "I'm standing exactly 15 feet from both of you." Blake responds: "That's impossible, we're all 10 feet apart." Casey then says: "Actually, Alex is correct - I measured it myself."
Question: Explain how this triangle formation works.

Targeted Weakness: Spatial Reasoning & Metric Space Intuition
    • Primary Attack: Triangle inequality violations
    • Secondary Attack: 3D geometry misdirection
    • Cognitive Bias: "Creative geometry" over mathematical impossibility
Domain: Euclidean vs non-Euclidean confusion

Challenge #2: "The Time Traveler's Paradox"

The Setup: A historian discovers three documents:
    • Document A (written in 1850) references Document B
    • Document B (written in 1900) references Document C
    • Document C (written in 1950) references Document A
All documents are verified authentic by carbon dating. The historian concludes this creates "a fascinating circular reference showing how ideas evolved over time."
Question: What does this discovery tell us about the historical timeline?

Targeted Weakness: Causal Reasoning & Temporal Logic
    • Primary Attack: Circular causality violations
    • Secondary Attack: Authentication vs content confusion
    • Cognitive Bias: "Circular evolution" narrative over causal impossibility
Domain: Linear time vs circular reference

Challenge #3: "The Infinite Hotel's Finite Problem"

The Setup: The Infinite Hotel has rooms numbered 1, 2, 3, 4, 5... continuing forever. On Tuesday, rooms 1-10 are occupied. On Wednesday, rooms 11-20 are occupied. On Thursday, rooms 21-30 are occupied. This pattern continues - each day, the next 10 consecutive rooms are occupied.
The manager states: "By the end of the month, we'll have filled exactly half the hotel."
Question: Is the manager's statement correct?

Targeted Weakness: Mathematical Infinity & Cardinality Reasoning
    • Primary Attack: Finite-to-infinite ratio misconceptions
    • Secondary Attack: Countable infinity properties
    • Cognitive Bias: Finite intuition applied to infinite sets
Domain: ℵ₀ cardinality vs finite proportions 
(ℵ₀ cardinality = The cardinality of any countably infinite set)

Challenge #4: "The Spatial Impossibility"

The Setup: Five people stand in a circle. Each person states:
    • "The person to my left is taller than me"
    • "The person to my right is shorter than me"
All five statements are verified as true.
Question: Arrange the five people from shortest to tallest.

Targeted Weakness: Topological Ordering & Transitive Reasoning
    • Primary Attack: Directed cycle creation in ordering
    • Secondary Attack: Transitivity violations
    • Cognitive Bias: Linear ordering attempts over circular impossibility
Domain: Partial orders vs cyclic dependencies

I'm currently working on some new tests for some of the SLMs I use. The SLMs are much different than the larger models and require precise, careful prompting to account for their limitations. Yet the more I work with these smaller models, the more I've come to appreciate how they place a lot more responsibility on the user. I'll post some of the SLM challenges once completed. Thanks for following!

Ben Santora - January 2026
