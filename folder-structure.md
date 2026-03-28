# Shared Memory Folder Structure

This folder structure serves as the shared state layer for a multi-instance, multi-platform AI orchestration system. It uses Google Drive (or any cloud file storage) so that any instance with access can read from and write to a common location.

## Structure

```
/AI_Orchestration/
│
├── /summaries/
│   ├── [instance_name]_[YYYY-MM-DD].md
│   ├── [instance_name]_[YYYY-MM-DD].md
│   └── ...
│
├── /handoffs/
│   ├── [YYYY-MM-DD]_[from_instance]_[to_instance].md
│   └── ...
│
├── /memory/
│   ├── global_corrections.md
│   ├── active_projects.md
│   └── terminology.md
│
├── /artifacts/
│   ├── [project_name]_[artifact_name].md
│   └── ...
│
└── /dispatch_logs/
    ├── [YYYY-MM-DD]_dispatch.md
    └── ...
```

## Folder Descriptions

### /summaries/
Session summaries produced by each instance at session end, following the [summary protocol](summary-protocol.md). This is the primary mechanism for context sharing. Each file is named with the instance name and date so any reader can quickly find the latest summary from any instance.

### /handoffs/
Directed context transfers between specific instances. Unlike summaries (which are available to everyone), handoffs are targeted: Instance A writes a document specifically for Instance B. Use handoffs when the context is only relevant to one recipient, or when it requires more detail than a summary provides.

### /memory/
Persistent shared state that applies across all instances and all sessions.

- **global_corrections.md:** Corrections that every instance needs to incorporate. When a fact, terminology choice, or preference is corrected in one instance, log it here so the dispatch agent can propagate it to all others.
- **active_projects.md:** Current project status across all instances. Updated when projects start, change scope, or complete.
- **terminology.md:** Agreed terminology decisions. Example: "Use 'multi-instance' not 'multi-agent' when describing the system."

### /artifacts/
Shared deliverables that any instance might need to reference. These are outputs of the work (documents, frameworks, analyses), not session metadata. Named by project and artifact for easy identification.

### /dispatch_logs/
Records of what the dispatch agent delivered, to whom, and when. The human reviews these at the start of each session to verify context was transferred correctly. Each log entry includes: what was delivered, target instance, timestamp, dispatch mode (event-driven or scheduled), and any errors encountered.

## Setup

1. Create the `/AI_Orchestration/` folder in your cloud file storage (Google Drive, Box, Dropbox, or equivalent)
2. Create each subfolder
3. Create starter files for `/memory/` (global_corrections.md, active_projects.md, terminology.md), even if they're initially empty
4. Share the folder with any service accounts or integrations that need access (e.g., MCP connectors)

## Naming Conventions

- Instance names: lowercase, no spaces (e.g., `strategy`, `advisory`, `observer`)
- Dates: `YYYY-MM-DD` format, always
- Projects: lowercase with underscores (e.g., `client_a_strategy`)
- Artifacts: descriptive, lowercase with underscores (e.g., `client_a_product_narrative_v5`)

---

*From the [Context Architecture](https://github.com/asanch3z/context-architecture) repo. Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
