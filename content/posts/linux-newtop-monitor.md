---
layout: default
title: "I Built My Own System Monitor — Because the Alternatives Were Too Fat"
description: "Why I wrote newtop, a minimal terminal system monitor in Go that reads directly from /proc and /sys — no dependencies, one binary, runs anywhere."
date: 2026-02-01
slug: "i-built-my-own-system-monitor"
author: "Ben Santora"
tags: ["Go", "Linux", "terminal", "tools", "system monitor", "htop", "proc"]
---

As a Linux user, there's a tool I've run for years called `htop`. It's a terminal-based system monitor — CPU usage, memory, running processes. It works. But it carries dependencies I don't need, and on a modest machine every dependency has a cost.

So I wrote my own. It's called `newtop`, it's written in Go, and it does exactly what I need and nothing else.

---

## The Problem With Modern Tooling

I run CrunchBang++ on an HP ENVY laptop. It's not a powerhouse. I work bare metal — no desktop environment, no compositor, no unnecessary services. Every CPU cycle I can reclaim is a cycle available for actual work: compiling, inference, processing.

The Linux ecosystem has a habit of solving simple problems with heavy tools — Windows and Mac are even worse. Need to monitor your system? Here's an Electron app. Need a terminal? Here's one wrapped in a browser engine. Nearly every Claude Code tutorial on YouTube I've seen features the instructor running Claude Code — a CLI tool — inside VS Code, which is itself an Electron application. If you love VS Code, fine; for me, it's like going out to the mailbox in a forklift.

My instinct goes the other way. If a terminal works, use the terminal. If Go produces a static binary with no runtime dependencies, use Go. If I can read directly from `/proc` and `/sys` without pulling in a library, I will.

---

## What newtop Does

It reads your system once per second and renders it directly in your terminal:

- Per-thread CPU bars with live frequency and temperature
- Memory: used, available, cached
- Disk I/O: read and write rates per physical device
- Network: RX and TX per interface
- Top processes by CPU usage, scaled to fit your terminal height

It auto-detects your hardware at startup — thread count, CPU model, terminal dimensions. Resize your terminal window and it adapts on the next tick. On wide terminals it goes two-column for the process list.

No config file. No setup. One binary, drop it anywhere, run it.

![newtop running on Linux](linux-newtop.png)

---

## Why Go

Go fits this kind of tool well. It compiles to a single static binary — copy it to any Linux machine and run it without installing anything. The standard library covers everything newtop needs: file I/O, string parsing, system calls. No external packages.

The binary is small. It starts instantly. When you close it, it's gone — no background processes, no lingering RAM.

That's the whole point.

---

## The Interesting Technical Bit

The part I'm most satisfied with is the CPU percentage calculation. Getting it wrong is easy — a common mistake assumes the loop runs in exactly one second. It doesn't, and on a busy system that error accumulates. The result is numbers technically derived from `/proc` but meaningless in practice.

The correct method, which matches what `htop` and `top` use:

```
cpu% = (delta_ticks / USER_HZ / elapsed_seconds) × 100
```

Where `delta_ticks` is the change in user + system time from `/proc/[pid]/stat` between samples, and `USER_HZ` is the Linux clock tick rate (100 on virtually all systems). Divide by elapsed real time, not by an assumed one-second interval — because the loop doesn't run in exactly one second, and on a loaded system the drift matters.

It sounds simple. But get it wrong and your process list shows 340% CPU for a sleeping browser tab.

---

## Get It

Clone it, run `go build`, drop the binary on your PATH.

[github.com/bensantora-tech/linux-newtop](https://github.com/bensantora-tech/linux-newtop)

No stars, no badges, no package registry. Just a tool that works on Linux, reads from `/proc`, and gets out of your way.

---

*Ben Santora — independent, bare metal Linux, Go programming.*
