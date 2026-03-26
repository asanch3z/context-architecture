# Context Architecture: Cross-Platform AI Orchestration for Knowledge Work

Knowledge workers who operate multiple AI instances across different platforms hit a coordination ceiling: context doesn't flow between tools, corrections don't propagate, and the human becomes the bottleneck of their own system. This repo documents a working architecture for solving that problem.

## What This Is

A methodology (not software) for orchestrating multiple AI instances across platforms like Claude, OpenAI, and browser-based agents, with shared memory, structured handoff protocols, and automated dispatch. Everything here is immediately usable with zero installation.

The full spec is a product requirements document covering the problem, opportunity, solution architecture, system diagrams, dispatch workflows, a summary protocol template, prioritized user stories with acceptance criteria, and metrics with guardrails.

## Who This Is For

Consultants, strategists, analysts, researchers, and other knowledge workers who have moved past single-chat AI usage and are running multiple instances with different roles across different tools. If you've built your own system by hand and you're spending increasing time routing context between platforms, this is for you.

## Why This Matters

The entire multi-agent tooling landscape (Claude Code Agent Teams, Gas Town, Multiclaude, Ruflo) is built for software development: agents writing code, running tests, reviewing PRs. Nobody has published a working system for non-engineering knowledge work. The people who are furthest ahead in using AI for knowledge work are also the most constrained by the lack of coordination infrastructure. This repo is a first attempt at defining that infrastructure.

## What's Inside

| File | Description |
|------|-------------|
| spec.md | Full product spec (v0.2): problem, opportunity, solution, system diagram, dispatch workflow, user stories, metrics |
| summary-protocol.md | Standalone template for structured session summaries that any instance can produce and any other instance can consume |
| folder-structure.md | Specification for the shared memory layer (Google Drive folder organization) |

## Core Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  HUMAN (Orchestrator)                     │
│         Designs workflows, triggers dispatch,            │
│         reviews output, holds quality gate                │
└──────────┬──────────────┬──────────────┬────────────────┘
           │              │              │
           ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐
│  Instance A  │ │  Instance B  │ │  Instance C          │
│  (Claude)    │ │  (OpenAI)   │ │  (Browser Agent)     │
│              │ │              │ │                      │
│  Strategy,   │ │  Advisory,   │ │  Observation,        │
│  analysis,   │ │  tone,       │ │  screen capture,     │
│  documents   │ │  comms       │ │  DISPATCH AGENT      │
└──────┬───────┘ └──────┬───────┘ └──────┬───────────────┘
       │                │                │
       │    writes      │    writes      │   reads/writes
       │    summary     │    summary     │   + delivers
       ▼                ▼                ▼
┌─────────────────────────────────────────────────────────┐
│              GOOGLE DRIVE (Shared Memory)                 │
│                                                          │
│  /summaries/     Session summaries from each instance    │
│  /handoffs/      Directed context transfers              │
│  /memory/        Global corrections, terminology,        │
│                  active projects                         │
│  /artifacts/     Shared deliverables                     │
└──────────────────────────────────────────────────────────┘
```

The human is always the orchestrator, decision-maker, and quality gate. The infrastructure handles context transfer so the human can focus on judgment.

## Key Principles

**Human-directed, not autonomous.** This is not a multi-agent system where AI agents coordinate independently. The human designs every workflow, verifies every output, and makes every strategic decision. The system handles the plumbing.

**Protocol-driven, not ad hoc.** Structured summary templates ensure every instance produces context in a consistent, consumable format. The protocol is what makes the system reliable, not the tools.

**Platform-agnostic.** The architecture works across any combination of AI platforms. The shared memory layer (Google Drive) and the dispatch agent (browser automation) are the integration points. No vendor lock-in.

**Context fidelity over speed.** The goal is maintaining the quality of context as it moves between instances, not just moving it faster. Every transfer preserves the information that makes AI output useful.

## How to Use This

1. Read the spec to understand the architecture
2. Copy the summary protocol template and start using it at the end of your AI sessions
3. Create the folder structure in your Google Drive
4. Start with manual dispatch (you deliver summaries between instances) and evaluate whether the structure improves your workflow before automating anything

You don't need to implement everything at once. The summary protocol alone will improve context quality across your instances.

## Status

This is a working draft (v0.2), published to establish the pattern and invite iteration. The architecture is based on a system the author has been operating in production for consulting work since late 2024. The spec will evolve as the proof of concept develops.

## Author

Alex Sanchez is a senior product leader with 15+ years of experience across mobile, AI/ML, marketplaces, and conversational AI (LinkedIn, Coursera, SmartThings, Wish). He currently runs an AI-augmented consulting practice built on the multi-instance, multi-platform system this spec documents.

[LinkedIn](https://www.linkedin.com/in/alexsanchez) | [Resume available on request]

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You're free to share and adapt it for any purpose, including commercial use, as long as you provide attribution.
