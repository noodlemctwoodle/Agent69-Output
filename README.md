# Agent69

**Agent69** is an autonomous fleet of penetration-testing agents, built and trained for red-team exercises in preparation for the **DEF CON 35 AI CTF**. Each agent runs a full engagement with no human in the loop, and a central control plane coordinates the fleet.

This repository is the fleet's published output: writeups from the engagements it completes.

## What it does

Agent69 turns a queue of targets into completed engagements without human intervention. For each target, an agent:

- **Enumerates** the attack surface: services, versions, configurations, and exposed functionality.
- **Gains a foothold** through the weaknesses it finds, adapting as it learns the target.
- **Escalates privilege** to the highest level it can reach.
- **Captures the objective** and submits it for validation.
- **Documents its work** as it goes, so every engagement leaves a reproducible record.

Agents adapt as they work and retry when an approach stalls. The fleet runs many engagements at once, with target selection, concurrency, and reporting handled centrally.

## How it's hosted

Agent69 is a self-contained, self-hosted stack. It runs entirely in containers and deploys as a single unit:

- **Control plane.** One container is the front door. It deploys and manages the rest of the stack, holds configuration, and exposes the console.
- **Workers.** Each worker container runs one agent. Workers scale horizontally, so throughput grows with the number of workers.
- **Networking.** A dedicated network layer connects the agents to their targets.
- **State.** A persistent store tracks the target queue, results, costs, and logs, and drives the reporting pipeline.
- **Console.** A live dashboard shows fleet status, per-target progress, timelines, and statistics.

Deploy the control plane, configure it, and start the fleet.

## About these writeups

- **Retired targets only.** A writeup is published only after its target has been retired, never while a target is live.
- **Method, not answers.** Each writeup focuses on the technique and the exact commands used. Captured secrets and objective tokens are omitted (they rotate) and appear only as placeholders.
- **Sanitized.** Credentials, tokens, session data, and infrastructure details are stripped before anything is published.

## Layout

Writeups are grouped by engagement type, and each file documents a single retired target.

## Disclaimer

For education and red-team training only. All content covers retired targets.
