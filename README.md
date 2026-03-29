# Context Architecture: Cross-Platform AI Orchestration for Knowledge Work

Knowledge workers who operate multiple AI instances across different platforms hit a coordination ceiling: context doesn't flow between tools, corrections don't propagate, and the human becomes the bottleneck of their own system. This repo documents a working architecture for solving that problem, built around the **Context Sync Protocol**: a standardized way for AI instances across different platforms to share, update, and compound context over time.

## What This Is

A methodology (not software) for orchestrating multiple AI instances across platforms like Claude, OpenAI, and browser-based agents, with shared memory, structured handoff protocols, and automated dispatch. Dispatch can be event-driven (triggered after a session) or scheduled on a recurring cadence for background consistency. Everything here is immediately usable with zero installation.

At its core is the **Context Sync Protocol**: think of it as MCP (Model Context Protocol) for memory state rather than tool access. MCP gives a model access to external tools. The Context Sync Protocol gives a model access to the evolving context of a multi-instance system.

The full spec is a product requirements document covering the problem, opportunity, solution architecture, system diagrams, dispatch workflows, a summary protocol template, prioritized user stories with acceptance criteria, metrics with guardrails, competitive positioning, and an evolution roadmap.

## Who This Is For

Product managers, consultants, strategists, analysts, researchers, and other knowledge workers who have moved past single-chat AI usage and are running multiple instances with different roles across different tools. If you've built your own system by hand and you're spending increasing time routing context between platforms, this is for you.

## Why This Matters

The entire multi-agent tooling landscape (Claude Code Agent Teams, Gas Town, Multiclaude, Ruflo) is built for software development: agents writing code, running tests, reviewing PRs. Multi-model tools like MultipleChat solve answer verification by sending the same prompt to multiple models and merging responses. Neither addresses the problem of sustained context coordination across ongoing work.

While orchestration frameworks exist for coding agents on single platforms, no widely adopted system addresses sustained context coordination across multiple platforms for knowledge work. This repo is a first attempt at defining that infrastructure.

## How This Differs from Multi-Model Tools

Tools like MultipleChat send one prompt to four models and merge the best answer. That's valuable for answer verification. Context Architecture solves a different problem: keeping specialized instances coordinated on an evolving body of work over time.

The key difference: multi-model tools are stateless per prompt. Context Architecture is a **closed-loop learning system** where context compounds. Every session consumes context from a shared store, does its work, and writes new context back. The store gets richer with every cycle. The system doesn't just coordinate; it learns.

## What's Inside

| File | Description |
|------|-------------|
| [spec.md](spec.md) | Full product spec (v0.4): problem, opportunity, solution, Context Sync Protocol, dispatch modes, system diagram, dispatch workflow, user stories, metrics, competitive positioning, evolution roadmap |
| [summary-protocol.md](summary-protocol.md) | Standalone template for structured session summaries that any instance can produce and any other instance can consume |
| [folder-structure.md](folder-structure.md) | Specification for the shared memory layer (Google Drive folder organization) |

## Core Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  HUMAN (Orchestrator)                   │
│         Designs workflows, triggers dispatch,           │
│         reviews output, holds quality gate              │
└──────────┬──────────────┬──────────────┬────────────────┘
           │              │              │
           ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐
│  Instance A  │ │  Instance B  │ │  Instance C          │
│  (Claude)    │ │  (OpenAI)    │ │  (Browser Agent)     │
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
│              SHARED MEMORY (Context Store)              │
│                                                         │
│  /summaries/     Session summaries from each instance   │
│  /handoffs/      Directed context transfers             │
│  /memory/        Global corrections, terminology,       │
│                  active projects                        │
│  /artifacts/     Shared deliverables                    │
└─────────────────────────┬───────────────────────────────┘
                          │
                    Context compounds
                    with every cycle
```

The human is always the orchestrator, decision-maker, and quality gate. The infrastructure handles context transfer so the human can focus on judgment. The system gets smarter with every session.

## Key Principles

**Human-directed, not autonomous.** This is not a multi-agent system where AI agents coordinate independently. The human designs every workflow, verifies every output, and makes every strategic decision. The system handles the plumbing.

**Protocol-driven, not ad hoc.** The Context Sync Protocol ensures every instance produces context in a consistent, consumable format. The protocol is what makes the system reliable, not the tools.

**Platform-agnostic.** The architecture works across any combination of AI platforms. The shared memory layer and the dispatch agent are the integration points. No vendor lock-in.

**Context compounds, not just transfers.** Every session both consumes and produces context. The shared store gets richer with every cycle. This is a learning system, not just a coordination tool.

## Evolution Roadmap

**Layer 1 (current, POC):** File-based shared memory (Google Drive), summary protocols, browser automation for dispatch. Supports both event-driven dispatch (triggered after sessions) and scheduled dispatch (recurring cadence for background sync). Validates the pattern.

**Layer 2 (API middleware):** Universal context store (database), platform adapters inject context into API calls. Closed-loop where sessions consume and produce context. Memory injected at three levels (global, project, instance).

**Layer 3 (protocol standard):** Platforms natively support context sync. Endpoints for "what changed since last sync" and "incorporate this context update." Eliminates middleware.

**Layer 4 (product):** Full platform with context store, adapters, sync engine, and dashboard. Replaces multiple AI subscriptions with unified orchestration.

## How to Use This

1. Read the [spec](spec.md) to understand the architecture and the Context Sync Protocol
2. Copy the [summary protocol template](summary-protocol.md) and start using it at the end of your AI sessions
3. Create the [folder structure](folder-structure.md) in your Google Drive
4. Start with manual dispatch (you deliver summaries between instances) and evaluate whether the structure improves your workflow before automating anything

You don't need to implement everything at once. The summary protocol alone will improve context quality across your instances.

## Status

This is a working draft (v0.4), published to establish the pattern and invite iteration. The architecture is based on a system the author has been operating in production for consulting work since late 2024. The spec will evolve as the proof of concept develops.

## Author

**Alex Sanchez** is a senior product leader with 15+ years of experience across mobile, AI/ML, marketplaces, and conversational AI (LinkedIn, Coursera, SmartThings, Wish). He currently runs an AI-augmented consulting practice built on the multi-instance, multi-platform system this spec documents.

[LinkedIn](https://www.linkedin.com/in/alexandersanchez/) | [Resume available on request]

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You're free to share and adapt it for any purpose, including commercial use, as long as you provide attribution.
