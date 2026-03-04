# Lazarus Engine

> *Software should serve the machine, not the other way around.*

A zero-dependency hardware audit tool built in Go. Designed to run on machines that modern tooling has abandoned — broken package managers, ancient kernels, ARM routers, discarded NAS boxes. If it runs Linux, this runs on it.

---

## Why This Exists

Modern software creates a **Dependency Death Spiral**: you can't run the app without the library, you can't update the library without the OS, and you can't update the OS without buying new hardware. Billions of functional machines end up in landfills because of software rot, not hardware failure.

The Lazarus Engine breaks that chain. A single static binary, no shared libraries, no runtime dependencies — just direct syscalls to the Linux kernel.

---

## What It Does

`lazarus-audit` probes the system and classifies hardware into one of three tiers:

| Status | Criteria | Recommended Path |
|--------|----------|-----------------|
| 🥇 GOLD | AES + AVX (or ARM equivalents) | Full modern FOSS stack |
| 🥈 SILVER | AES only | Most tasks; AVX workloads may struggle |
| 🥉 BRONZE | Neither | Minimalist tools: ffplay, aplay, busybox |

**Sample output:**
```
--- [LAZARUS ENGINE: HARDWARE AUDIT] ---
[CPU] Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz (amd64)
[RAM] 3.73 GB
[OS ] Linux Kernel 5.15.0-91-generic
----------------------------------------
[STATUS] GOLD: Modern instruction sets detected.
[ACTION] This machine can handle modern FOSS effortlessly.
```

---

## Build

### Standard build
```bash
go build -o lazarus-audit
```

### Extreme Shrink (recommended for constrained hardware)
```bash
# Strips symbol tables, debug info, and dynamic library linkage
CGO_ENABLED=0 go build -ldflags="-s -w" -trimpath -o lazarus-audit

# Optional: compress binary further (requires upx)
upx --brute lazarus-audit
```

The shrink pipeline targets machines with 2GB RAM or mechanical hard drives where binary size and disk I/O genuinely matter. Typical output: **~1.2 MB**.

### Cross-compilation (edge devices)
```bash
# ARM (Raspberry Pi, old routers)
GOOS=linux GOARCH=arm GOARM=6 CGO_ENABLED=0 go build -ldflags="-s -w" -o lazarus-audit-arm

# ARM64 (NAS boxes, newer SBCs)
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -ldflags="-s -w" -o lazarus-audit-arm64

# MIPS (many consumer routers)
GOOS=linux GOARCH=mips GOMIPS=softfloat CGO_ENABLED=0 go build -ldflags="-s -w" -o lazarus-audit-mips
```

---

## Architecture Support

The audit correctly handles both x86/x86_64 and ARM/MIPS flag formats:

- **x86/x86_64** — reads `flags` field from `/proc/cpuinfo`
- **ARM/MIPS** — reads `Features` field; maps `aes-ce`/`pmull` → AES tier, `asimdhp`/`sve` → AVX tier

This means the GOLD/SILVER/BRONZE classification is meaningful on the edge devices this tool is designed for, not just on desktops.

---

## The Mission: Resourceful Computing

This tool is part of a larger framework. The three-step restoration process:

1. **Audit** — run `lazarus-audit` to know what you're working with
2. **Strip** — remove the desktop environment; reclaim RAM
3. **Restore** — deploy static binaries to turn "junk" into a dedicated media server, VPN gateway, or network ad-blocker

Full writeup: [Binary Resurrection](https://github.com/bensantora-tech/lazarus-engine)

---

## Requirements

- Linux (any kernel version with `/proc` filesystem)
- Go 1.18+ to build
- No runtime dependencies

---

## License

GPLV-3

## Author

Ben Santora
