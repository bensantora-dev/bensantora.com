---
title: "Your Linux Kernel Has a Built-In Chaos Engine — And Nobody Told You"
date: 2026-03-07
description: "Linux has shipped a built-in network chaos simulator since kernel 2.6. No install required — tc netem lets you add latency, packet loss, and jitter directly in the kernel's network stack. Here's how to use it, and why you need to clean up after yourself."
tags: ["linux", "networking", "testing", "devops", "go"]
---

Your program works perfectly on your machine. Fast responses, clean connections, zero errors. You ship it.

Then someone runs it from a hotel WiFi in rural Ohio. Or from Brazil over a satellite link. Or from a corporate network that silently drops one in twenty packets. Your program falls apart — timeouts unhandled, no retry logic, error messages that say "something went wrong" and nothing else.

You never saw it coming because your test environment was perfect. That's the bug.

---

## The Capability You Already Have

Here's what most Linux developers don't know: the kernel has been able to simulate a terrible network since the early 2000s. No tools to install. No Docker container. No external service. It's sitting on your system right now, part of `iproute2`, which ships on virtually every Linux distribution.

It's called **netem** — network emulator — and it's accessed through a two-letter command: `tc`.

`tc` stands for *traffic control*. Its job is managing the queuing disciplines — qdiscs — that govern how packets move through your network interfaces. By default your interface uses a simple qdisc that sends packets out as fast as possible in order. netem replaces that default with something that intercepts every outgoing packet and does something deliberate to it before it hits the wire.

```bash
# Add 200ms delay with 20ms jitter to outgoing traffic
sudo tc qdisc add dev eth0 root netem delay 200ms 20ms

# Simulate 5% packet loss
sudo tc qdisc add dev eth0 root netem loss 5%

# Both at once — bad wifi in two lines
sudo tc qdisc add dev eth0 root netem delay 200ms 20ms loss 1%

# Simulate a satellite connection
sudo tc qdisc add dev eth0 root netem delay 600ms 50ms loss 2%

# Remove it — restore normal behavior
sudo tc qdisc del dev eth0 root
```

That's it. No install. No config file. No daemon. You just told the kernel to wreck your own network, and the kernel said fine.

This is not a proxy sitting in front of your traffic. It's not a simulation layer. It's happening inside the kernel's actual network stack, on real packets, on a real interface. When you curl an API with 600ms of latency applied, you are experiencing what your users in bad network conditions experience. Exactly.

---

## What You Can Actually Test

The value isn't in the novelty — it's in what you find when you use it.

**Timeout handling.** Set 5 seconds of delay and watch whether your application waits forever or fails gracefully. Most don't fail gracefully the first time.

**Retry logic.** Apply 10% packet loss and see whether your program retries, gives up immediately, or enters some undefined state where it thinks it succeeded but didn't.

**Connection resilience.** Apply loss mid-session and see whether long-lived connections recover or require a restart.

**User-facing error messages.** Under bad conditions, what does your application actually tell the user? "Error" is not an answer.

These are not edge cases. These are conditions that a real percentage of your users experience on any given day. Testing against them locally, before shipping, costs nothing if the tooling is already there.

---

## The Part Nobody Mentions

Here's where it gets important.

netem qdiscs are **persistent kernel state**. They live on the network interface itself — not in your terminal session, not in your shell environment, not attached to the process that created them. When you close your terminal, the rule stays. When you log out, it stays. When you kill the shell, it stays.

The kernel has no concept of "the session that applied this." It just knows the interface has a qdisc, and it applies it to every packet, indefinitely, until something explicitly removes it.

The only thing that clears it automatically is a **full reboot**, because the kernel reinitializes network interfaces from scratch on boot.

This matters in practice. Developers who use `tc netem` directly — usually by copying a command from Stack Overflow — frequently forget to run the `del` command. They finish their test, close the terminal, get on with their day. An hour later their machine feels slow. Their SSH sessions are laggy. Downloads are crawling. They restart their browser. They check their router. They don't think to check `tc qdisc show dev eth0`, because why would they? Nothing should still be running.

But it is.

You can inspect what's currently applied at any time:

```bash
tc qdisc show dev eth0
```

On a clean interface you'll see something like `qdisc fq_codel` or `qdisc pfifo_fast` — the default. If you see `qdisc netem`, you left something behind.

---

## Working With It Safely

A few habits that save you from the forgotten-rule problem:

**Always have the delete command ready before you run the add command.** Write both in your terminal history in the same session. Don't run the add and close the terminal.

**Alias the cleanup.** Put this in your `.bashrc` or `.bash_aliases`:

```bash
alias tc-clear='sudo tc qdisc del dev eth0 root 2>/dev/null; echo "cleared"'
```

Adjust the interface name for your system (`ip link` shows what you have). Now cleanup is one word.

**Check before you test.** Make it a habit to run `tc qdisc show` before and after any network chaos session. Costs two seconds, saves an afternoon of confusion.

**Know your interface name.** `eth0` is traditional but your system might use `enp3s0`, `wlan0`, `ens33`, or something else entirely. `ip link` shows all interfaces and their current state. Using the wrong interface name means your rules silently apply to the wrong place — or fail with a cryptic error.

---

## The Bigger Point

The Linux kernel is full of capabilities like this — powerful, precise, already present, largely invisible to the average developer because nobody put a friendly face on them. `tc netem` has been shipping on Linux systems since kernel 2.6. Most of the developers who would benefit most from it have never heard of it.

The barrier wasn't installation. It wasn't licensing. It wasn't compatibility. It was just awareness.

Now you know it's there. Use it deliberately, clean up after yourself, and go find the bugs your users were going to find for you.

---

*Next up: building a small Go tool that wraps `tc netem` with named profiles, state tracking, and guaranteed cleanup — so you never leave your network broken by accident.*

---

