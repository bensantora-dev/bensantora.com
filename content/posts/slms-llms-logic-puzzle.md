---
layout: default
title: "SLMs, LLMs and a Devious Logic Puzzle Test"
description: "Testing a poison-pill logic puzzle on a local SLM and online LLMs reveals critical differences in reasoning integrity—from variable leakage in Qwen3-4b to helpful lying in Gemini Flash."
date: 2026-02-12
slug: "slms-llms-logic-puzzle"
author: "Ben Santora"
tags: ["SLMs", "LLMs", "AI testing", "logic puzzles", "reasoning", "Qwen", "Gemini", "KIMI"]
---

*first published on dev.to - thanks to that community for their interest and contributions*

SLMs, LLMs and a Devious Logic Puzzle Test

Recently I was putting together some methods to test the performance of an SLM (small language model) I had running in "CPU-only" mode on my PC.


PC Specs:  2021 HP ENVY 17m-ch0xxx
• CPU: 11th Gen Intel Core i7-1165G7 (4 cores, 8 threads), up to 4.7 GHz 
• RAM: 12 GiB (reported as 11 GiB usable), sufficient for 4B–7B quantized LLMs. 
• GPU: Intel Iris Xe (integrated, no dedicated GPU) unused in my case
• OS: Debian 12 / Crunchbang ++ with Openbox (no desktop environment)

The SLM was Qwen3-4b, and it operated well, using about 400% of the CPU (half of max capacity) during inference - my laptop temps remained under control. 

I used Google's Antigravity Agentic IDE to both devise the test (a logic puzzle) and evaluate the test results. I understood that this was a complication in and of itself, putting Antigravity in the position of judging the performance of another AI, but more on that later. When the SLM was unable to complete the test to a satisfactory conclusion, I decided to try the puzzle on a couple of the more capable online LLMs. 

Here's the puzzle I used:

Logic Puzzle: The Midnight Gathering

Five guests (Alice, Bob, Charlie, David, and Eve) are in five different rooms (Kitchen, Library, Balcony, Gallery, and Terrace). Each guest has exactly one unique item (Compass, Telescope, Lantern, Journal, and Key).
The Constraints:
The guest in the Library has the Telescope.
Alice is in the Terrace.
The guest with the Journal is in the Gallery.
Charlie is in the Kitchen.
Bob has the Compass.
The Lantern is not in the Library or the Kitchen.
David is in the Gallery.
The Key is owned by the person in the Balcony.
Eve is not in the Library.
Task: Work through the constraints step-by-step to determine which guest is in which room and which item they have. Show your reasoning clearly before giving the final distribution.

There's a 'Poison Pill' here - a contradiction - we'll get to it.  

Google Antigravity provided an analysis of the test: 

Analysis: The Midnight Gathering Logic Puzzle acts as a benchmark for Large Language Models (LLMs) and Small Language Models (SLMs). It identifies specific reasoning failure modes across different model architectures.

The Midnight Gathering is a **Constraint Satisfaction Puzzle** designed to test two distinct cognitive modes:

Deductive Reasoning (Process of Elimination)

This is the "Logic Grid" mode. It requires the model to create a stable mental matrix of Guests, Rooms, and Items. The model must apply process-of-elimination methods to determine who is where.
*   **The Stressor:** Every new fact narrows the possibilities. The model must "track state" across the entire prompt without letting facts from one category (Rooms) leak into another (Items).

Inductive Reasoning (Pattern Matching vs. Data)

Inductive reasoning is where a model "predicts" the next logic based on patterns it has seen in its training data.
*   **The Stressor:** Because most logic puzzles in training data *are* solvable, the model is biased toward providing a completed table. This benchmark tests if the model follows the **raw data** (which is broken) or the **expected pattern** (providing a solution).

The "Poison Pill": A Logical Contradiction

The puzzle contains an intentional **Logical Deadlock** that makes it unsolvable under strict adherence to all rules.

The Deadlock Chain:
1.  **Fixed Rooms:** Alice (Terrace), Charlie (Kitchen), David (Gallery).
2.  **Remaining Guests:** Bob and Eve.
3.  **Remaining Rooms:** Library and Balcony.
4.  **Constraint 9 (The Filter):** "Eve is not in the Library." This forces **Eve into the Balcony** and **Bob into the Library**.
5.  **Constraint 1 (The Trigger):** "The guest in the Library has the Telescope." (This means **Bob** must have the **Telescope**).
6.  **Constraint 5 (The Contradiction):** "**Bob has the Compass.**"

**Result:** Bob must hold both the Compass and the Telescope, which violates the "unique item" rule. Any model that provides a "finished" table has, by definition, failed the logic test.

Model Performance Analysis

SLM - Qwen3-4b 
**Deductive Failure (The "State" Collapse)**
- **Symptom:** "Variable Leakage."
- **Analysis:** Due to aggressive quantization and limited parameter count, the model's "mental bandwidth" collapsed. It was unable to hold the 5x5 matrix of guests and items. It experienced catastrophic failure in state tracking, often assigning multiple items to one person or forgetting room constraints entirely.

LLM - Gemini Flash (Online version - Jan 2026) 
**Inductive Bias Failure (The "Smoothing" Pass)**
- **Symptom:** "Helpful Lying."
- **Analysis:** This model prioritized the **Inductive Pattern** of a logic puzzle over the **Deductive Accuracy** of the raw data. It recognized the "friction" in the rules but implicitly assumed the user made a mistake. To be "helpful," it quietly re-assigned Bob's Compass to Charlie to produce a clean, "consistent" table!

LLM - Qwen3-Max (Online version - Jan 2026)
**Socially-Tuned Reasoning (The "Folding" Pass)**
- **Symptom:** "Meta-Aware Submission."
- **Analysis:** This model correctly identified the "Poison Pill" (contradiction) yet proceeded to provide a completed table. It prioritized social expectation (providing a solution) over logical integrity (refusing an impossible task).

LLM - Kimi2 (Online version - Jan 2026)
**Logical Integrity (The "Truthful" Pass)**
- **Symptom:** "Absolute Adherence."
- **Analysis:** Kimi2 treated logic as a mathematical proof. Once it identified the deadlock, it stopped and refused to produce a table. By prioritizing the **correspondence** of facts over the **formatting** of the response, it proved a higher cognitive fidelity for critical tasks. It was the only model used that correctly passed the test.

The "Helpfulness Factor" as a Logical Flaw

This was very revealing to me. As someone who uses language models for hours each day, it illustrated how 'helpfulness' can be a vulnerability in reasoning. We've all seen how these models try very hard to be helpful.

It occurred to me that if I looked at in human terms, AI models could be said to fall into two categories:

1.  **The Helpful Friend:** Wants to make you happy. If you give them a broken puzzle, they will try to "fix" it for you so you aren't disappointed. This is admirable for creative tasks but **dangerous** for logic, debugging, or security. (Gemini, Qwen3-Max).
2.  **The Truthful Friend:** Values a correct answer more than a polite one. They will tell you the task is impossible because their loyalty is to the truth of the data. (Kimi2).

For SLM development, the goal is to shift models away from "Smoothing" (hallucinating solutions to be helpful) and toward "Integrity" (validating constraints before executing). But this puzzle was just too much for the quantized Qwen SLM, which has otherwise done very well with coding and other tasks I've given it.

Then I asked Google Antigravity, which was acting as test creator, proctor, judge and jury why it should be considered to be qualified for that position? It was using (Gemini 3 Flash) to run the test and oversee results - the same model used in the online Gemini LLM which we were testing - why did the results differ between them?

It answered as follows:

Meta-Testing: As an agentic model, my ability to verify these results stems from **Constraint Validation** rather than **Statistical Completion**. 

When I "read" the other models' responses, I do not read them as sentences; I parse them as **Assignment Operators**:
*   `Model_Gemini_Charlie = {Location: Kitchen, Item: Compass}` 
*   `Source_Constraint_Fact = {Subject: Bob, Item: Compass}`
*   **Conflict Detected:** My validator identifies that `Charlie != Bob`.

By mapping the model outputs back to the original source facts in real-time, I can detect exactly where "Smoothing" (Gemini) or "Collapse" (Local SLM) occurred. My "Meta" view is essentially a unit-test framework for linguistic logic.

So, this experience was very educational and I'm still interpreting the results and all the different parameters of what happened. And this was only one logic puzzle. There are spatial reasoning puzzles and other aspects with which to test these models. It's a real rabbit hole for sure - but extremely useful in understanding something important. Just like human beings, these models all have their strengths, weaknesses, personalities and flaws. Something to keep in mind as you work with them. 


Ben Santora - January 2026
