# Cross-Platform AI Orchestration for Knowledge Work
## Product Spec v0.4

**Author:** Alex Sanchez
**Date:** March 26, 2026
**Status:** Working draft, published for public iteration

---

## Problem

Knowledge workers who operate multiple AI instances across different platforms (Claude, OpenAI, browser-based agents, and others) hit a coordination ceiling that erodes the time savings AI provides. Context doesn't flow between platforms. Corrections made in one instance don't propagate to others. Every handoff requires the human to write a summary, decide what the receiving instance needs, reformat it as a prompt, and deliver it manually. The more specialized the system becomes, the more time the human spends routing rather than thinking. In observed practice, this manifests as 5-10 page handoff documents written at thread boundaries, terminology corrections that resurface across instances because they weren't propagated, session starts consumed by re-establishing context that was already developed elsewhere, and output quality that degrades whenever context transfer loses fidelity. The AI makes the work better. The coordination overhead makes the human the bottleneck of their own system.

## Opportunity

Nobody has built coordination infrastructure for multi-instance, multi-platform AI workflows outside of software development. The entire multi-agent tooling landscape (Claude Code Agent Teams, Gas Town, Multiclaude, Ruflo) is built for engineering: agents writing code, running tests, reviewing PRs. Knowledge workers (consultants, strategists, analysts, researchers) who have moved past single-chat AI usage have no equivalent. They're building their own systems by hand, which means the ones who are furthest ahead are also the ones most constrained by the lack of infrastructure. A lightweight coordination layer using existing tools (cloud file storage for shared state, browser automation for dispatch, structured protocols for summaries) could demonstrate the architecture without requiring new infrastructure, serve as a proof of concept for a broader product, and establish a reference pattern for a space where everyone is talking about context but nobody has shipped a working system for non-engineering knowledge work.

## Solution

A human-directed orchestration system where the infrastructure handles context transfer, state management, and cross-platform coordination, while the human retains all strategic decisions, quality gates, and prompt design. Three components:

**Shared Memory Layer (Cloud File Storage):** A structured folder in any cloud file storage service (Google Drive, Box, Dropbox, or equivalent) that serves as the system's shared state. Every instance writes session summaries to a common location. Every instance can read what other instances have produced. The folder structure encodes what type of information lives where: summaries, directed handoffs, global corrections, shared artifacts.

**Summary Protocol:** A standardized format that every instance follows at session end. The protocol defines what gets captured (what happened, decisions made, open items, context needed by other instances, corrections to shared memory) so that summaries are machine-readable and consistently structured, regardless of which platform or instance produced them.

**Dispatch Agent (Browser Automation):** A browser automation agent that reads new summaries from the shared memory layer and delivers relevant context to target instances across platforms. The agent navigates to the shared drive, identifies what's new, then navigates to the target platform and delivers the context via the platform's native input. The human triggers the dispatch cycle and reviews the dispatch log. The dispatch agent does not make decisions about what to deliver; the summary protocol and the human's configuration determine that.

**Dispatch Modes:** The dispatch mechanism supports two operational modes:

- **Event-driven dispatch:** Triggered when a session ends and a new summary is written to the shared memory layer. The human initiates the dispatch cycle, and the agent delivers context that has accumulated since the last dispatch. This is the primary mode for the MVP.

- **Scheduled dispatch:** The dispatch agent runs on a defined cadence (e.g., every few hours, three times per day, or at fixed times) regardless of whether a new session has occurred. Scheduled dispatch catches updates that event-driven dispatch might miss: manual edits to shared memory files, corrections added outside a formal session, or summaries written by instances that don't trigger dispatch directly. It also ensures that context stays fresh across instances even during gaps between active sessions.

Both modes write to `/dispatch_logs/` and are subject to the same human review. In practice, the two modes are complementary: event-driven dispatch handles immediate context flow after active sessions, while scheduled dispatch provides a background consistency layer.

---

## System Diagram

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
│              GOOGLE DRIVE (Shared Memory)               │
│                                                         │
│  /summaries/     Session summaries from each instance   │
│  /handoffs/      Directed context transfers             │
│  /memory/        Global corrections, terminology,       │
│                  active projects                        │
│  /artifacts/     Shared deliverables                    │
└─────────────────────────────────────────────────────────┘
```

## Dispatch Workflow

```
Event-driven dispatch:
1. Human completes a session in any instance
2. Instance writes summary per protocol to shared memory (cloud file storage)
3. Human triggers dispatch
4. Dispatch agent reads new summaries from /summaries/
5. For each summary with cross-instance context:
   a. Dispatch agent navigates to target platform
   b. Delivers context via native input
   c. Captures acknowledgment
   d. Writes dispatch log to shared memory
6. Human reviews dispatch log at next session start

Scheduled dispatch:
1. Dispatch agent runs at a defined cadence (e.g., 3x daily)
2. Agent scans /summaries/, /memory/, and /handoffs/ for changes since last sync
3. For each change with cross-instance relevance:
   a-d. Same delivery steps as event-driven
4. Agent writes dispatch log noting scheduled (not event-driven) trigger
5. Human reviews dispatch log at next session start
```

---

## Folder Structure

```
/AI_Orchestration/
  /summaries/
    [instance_name]_[date].md
  /handoffs/
    [date]_[from]_[to].md
  /memory/
    global_corrections.md
    active_projects.md
    terminology.md
  /artifacts/
    [project]_[artifact_name].md
  /dispatch_logs/
    [date]_dispatch.md
```

See [folder-structure.md](folder-structure.md) for the full specification.

---

## Summary Protocol

See [summary-protocol.md](summary-protocol.md) for the standalone template.

```markdown
# Session Summary: [Instance Name]
## Date: [date]
## Project: [project name]
## Duration: [approximate]

### What happened
[2-3 sentences]

### Key decisions made
- [decision 1]
- [decision 2]

### Open items
- [item 1]
- [item 2]

### Context for other instances
- [Target instance]: [what they need to know and why]

### Corrections to shared memory
- [correction, if any, with which file in /memory/ to update]

### Source verification
- [For factual claims introduced this session: sourced, inferred, or unverified]
```

---

## User Stories (Prioritized)

### P0: Must have for MVP

**US-1: Write summary to shared memory**
As a knowledge worker finishing an AI session, I want the instance to produce a structured summary and save it to the shared memory layer, so that the context from this session is available to other instances without me rewriting it.

*Acceptance criteria:*
- Summary follows the protocol template
- Summary is saved to /summaries/ with instance name and date
- Summary is readable by any instance with access to the shared memory layer
- Time to produce summary: under 2 minutes

**US-2: Read summary from shared memory**
As a knowledge worker starting an AI session, I want the instance to read the latest summaries from other instances, so that I don't spend the first 10 minutes of every session re-establishing context that already exists.

*Acceptance criteria:*
- Instance can read markdown files from /summaries/
- Instance incorporates relevant context without being explicitly prompted with the full content
- Time to read and absorb: under 1 minute

**US-3: Dispatch context between platforms**
As a knowledge worker, I want the dispatch agent to deliver context from one platform to another, so that I don't have to manually copy-paste between platforms.

*Acceptance criteria:*
- Dispatch agent navigates to shared memory and reads new summaries
- Dispatch agent navigates to target platform and delivers context via native input
- Dispatch agent writes a dispatch log confirming delivery
- Total dispatch time for a single cross-platform transfer: under 5 minutes
- Context accuracy: no information loss vs. manual handoff

### P1: Important for usability

**US-4: Propagate corrections globally**
As a knowledge worker, I want a correction made in one instance to propagate to all other instances, so that I don't encounter the same error in a different context.

*Acceptance criteria:*
- Correction is written to /memory/global_corrections.md
- On next dispatch cycle, dispatch agent delivers the correction to all instances
- Each instance acknowledges and incorporates the correction

**US-5: Dispatch log review**
As a knowledge worker, I want to review what was delivered and to whom, so that I can verify context quality and catch errors before they compound.

*Acceptance criteria:*
- Dispatch log saved to /dispatch_logs/ with date
- Log includes: what was delivered, to which instance, timestamp, and any errors
- Readable in under 2 minutes

### P2: Nice to have

**US-6: Filtered dispatch**
As a knowledge worker, I want to specify which parts of a summary go to which instance, so that each instance only receives context relevant to its role.

*Acceptance criteria:*
- Summary protocol includes a "Context for other instances" field with target instance specified
- Dispatch agent delivers only the relevant sections to each target
- Irrelevant context is not delivered

**US-7: Bidirectional dispatch**
As a knowledge worker, I want the dispatch agent to capture output from a target instance after delivering context and write it back to shared memory, so that the response is available to all other instances.

*Acceptance criteria:*
- Dispatch agent delivers context, waits for response, captures output
- Output saved to /handoffs/ with source and target labeled
- Available for next dispatch cycle

---

## Metrics

### Primary metrics
- **Context transfer time:** Time from "session ends in Instance A" to "context is available in Instance B." Baseline (manual): 10-20 minutes. Target (MVP): under 5 minutes.
- **Context fidelity:** Percentage of key information that arrives intact at the receiving instance. Baseline (manual): ~90% (some loss in every rewrite). Target (MVP): 95%+.
- **Session start time:** Time from "open a new session" to "productive work begins." Baseline: 5-15 minutes re-establishing context. Target: under 2 minutes.

### Guardrail metrics
- **False context:** Information delivered to an instance that is inaccurate or outdated. Must be 0%. One false correction propagated globally would undermine trust in the entire system.
- **Dispatch errors:** Failed deliveries (agent can't navigate, platform UI changed, timeout). Acceptable for MVP: under 20% failure rate (human catches and retries). Production target: under 5%.
- **Human review time per dispatch:** Time spent reviewing the dispatch log. Must stay under 2 minutes. If review time grows, the system is adding overhead rather than removing it.

---

## MVP Scope

**In scope:**
- Shared memory folder structure (cloud file storage)
- Summary protocol document
- Strategy instance writes summaries to shared memory
- Dispatch agent reads from shared memory and delivers to advisory instance via browser
- Dispatch log written to shared memory
- Human triggers dispatch manually (event-driven mode)
- Scheduled dispatch on a defined cadence (e.g., 2-3x daily)

**Out of scope for MVP:**
- Advisory instance writing summaries back to Drive (manual for now)
- Bidirectional capture (US-7)
- Filtered dispatch (US-6)
- Error recovery protocols
- Multi-instance simultaneous delivery

**Time box:** 2-3 hours for the POC. If the core workflow (write summary, dispatch, verify delivery) doesn't work in that window, reassess before investing more time.

---

## Background

This architecture is based on a system the author has been operating in production for consulting work since late 2024. The system uses specialized AI instances across Claude (strategy and analysis), OpenAI (advisory and client communications), and Anthropic's Cowork platform (browser-based observation and automation), coordinated through manual context handoffs with structured protocols.

The spec documents what works, what doesn't, and what infrastructure would make the system more efficient. It is published as a working draft to establish the pattern and invite iteration from other knowledge workers building similar systems.

---

## The Context Sync Protocol

The coordination mechanism at the core of this architecture is the **Context Sync Protocol**: a standardized way for AI instances across different platforms to share, update, and compound context over time. Think of it as MCP (Model Context Protocol) for memory state rather than tool access. MCP gives a model access to external tools. The Context Sync Protocol gives a model access to the evolving context of a multi-instance system.

The protocol defines:
- **What gets captured** at session end (the summary protocol template)
- **Where it lives** (the shared memory layer with three levels: global, project, instance)
- **How it flows** (dispatch from shared memory to target instances)
- **How it compounds** (every session both consumes and produces context, updating the shared store)

The closed-loop design is what makes this a learning system, not just a coordination tool. Each session starts by consuming context from the shared store. The instance does its work with that context, making decisions, producing output, identifying corrections. At session end, the instance produces a comprehensive summary that flows back to the store, updating memory at whichever level is appropriate (global correction, project update, or instance-specific knowledge). Next time any instance starts a session, it pulls from the updated store. The context gets richer with every cycle. It compounds.

---

## How This Differs from Multi-Model Tools

Tools like MultipleChat, Magai, and TeamAI solve a different problem. They send the same prompt to multiple AI models (ChatGPT, Claude, Gemini, Grok) and merge, verify, or debate the responses to produce a single higher-quality answer. This is valuable for answer verification on single prompts.

Context Architecture solves a fundamentally different problem: sustained coordination across ongoing work.

| Dimension | Multi-model tools (e.g., MultipleChat) | Context Architecture |
|-----------|---------------------------------------|---------------------|
| Problem solved | Answer accuracy through model diversity | Long-term context coordination across platforms |
| Time horizon | Single prompt, stateless | Weeks or months of sustained engagement |
| Model roles | Interchangeable (all answer the same question) | Specialized (strategy, communications, observation) |
| Memory | None between sessions | Three-level persistent memory (global, project, instance) |
| Context | Resets per prompt | Accumulated, transferred, and compounded across sessions |
| Human role | Prompt author, picks the best answer | Orchestrator, quality gate, strategic decision-maker |
| Output | One verified answer | Sustained deliverable production across workstreams |
| Core mechanism | Fan-out and merge | Context Sync Protocol (closed-loop, compounding) |

These approaches are complementary, not competing. A knowledge worker could use a multi-model tool for quick research verification AND Context Architecture for managing their multi-instance system on a long-term engagement.

---

## Evolution Roadmap

**Layer 1 (current spec, POC):** Cloud file storage (Google Drive, Box, Dropbox, or equivalent) as shared state, summary protocols, browser automation for dispatch. File-based, validates the pattern. Buildable in hours with zero additional cost.

**Layer 2 (API middleware):** Universal context store (database), platform adapters inject context into API calls, closed-loop where each session both consumes and produces context. The store replaces the file-based layer as the persistence mechanism. Each platform adapter knows how to read from and write to its platform's native context mechanism. Context depth is maintained because memory is injected at all three levels (global, project, instance) on every API call, and summaries flow back to update the store.

**Layer 3 (protocol standard):** A Context Sync Protocol that platforms themselves support natively. Platforms expose endpoints for "here's what changed in this user's context since the last sync" and "here's a context update from another platform, incorporate it." This eliminates the need for middleware or browser automation. Requires industry adoption. One person defines the spec and advocates for it.

**Layer 4 (product):** A platform with universal context store, platform adapters, sync engine, and human dashboard. Subscription replaces multiple individual AI subscriptions with a unified orchestration layer. The business model works best at team and enterprise scale, where the coordination layer provides governance, consistency, and shared context across team members and AI instances.

---

## Changelog

- **v0.4 (March 27, 2026):** Added dispatch modes: event-driven and scheduled. Defined scheduled dispatch as a complement to event-driven dispatch for background context consistency. Updated dispatch workflow to document both modes. Moved scheduled dispatch into MVP scope.
- **v0.3 (March 26, 2026):** Named the Context Sync Protocol. Added closed-loop architecture insight (context compounds, system learns over time). Added competitive positioning vs. multi-model tools (MultipleChat, etc.). Added evolution roadmap (four layers from POC to product). Added "How This Differs" comparison table.
- **v0.2 (March 25, 2026):** Rewritten in PRD format. Problem grounded in observed workflow. Added system diagram, dispatch workflow, user stories with acceptance criteria, metrics with guardrails, MVP scope. Generalized for public consumption.
- **v0.1 (March 25, 2026):** Initial draft. Internal only.

---

*Published by Alex Sanchez. Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
