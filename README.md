# Agent69

**Agent69** is an autonomous fleet of penetration-testing agents, built and trained for red-team exercises in preparation for the **DEF CON 35 AI CTF**. Each agent runs a full engagement with no human in the loop, covering reconnaissance, initial access, privilege escalation, and objective capture, while a central control plane coordinates targets, concurrency, retries, and reporting across the fleet.

This repository is the fleet's published output: writeups from the engagements it completes.

## What it does

- **Autonomous engagements.** Each agent works a target end to end: it maps the attack surface, gains a foothold, escalates privilege, and captures the objective.
- **Fleet coordination.** A control plane assigns targets, manages concurrency and retries, tracks progress, and collects results.
- **Reproducible reporting.** Every engagement is recorded as a step-by-step chain (recon, foothold, escalation, objective) and turned into a clean, structured writeup.

## About these writeups

- **Retired targets only.** A writeup is published only after its target has been retired, never while a target is live.
- **Method, not answers.** Each writeup focuses on the technique and the exact commands used. Captured secrets and objective tokens are omitted (they rotate) and appear only as placeholders.
- **Sanitized.** Credentials, tokens, session data, and infrastructure details are stripped before anything is published.

## Layout

Writeups are grouped by engagement type, and each file documents a single retired target.

## Disclaimer

For education and red-team training only. All content covers retired targets.
