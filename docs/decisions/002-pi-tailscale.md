# ADR 002 — Raspberry Pi + Tailscale for Hosting

**Status:** Accepted

## Context
A Raspberry Pi already runs on the home network for irrigation automation. The app needs to be reachable from both phones when away from home.

## Decision
Host on the existing Pi. Use Tailscale for secure remote access without port forwarding or public exposure.

## Consequences
- Zero additional hosting cost
- Data stays entirely self-hosted and private
- Both phones and Pi join the same Tailscale network
- Pi is a dependency — acceptable for a two-person private app
- Cloudflare Tunnel remains an option if broader access is ever needed
