# Summary Protocol Template

Use this template at the end of every AI session to produce a structured summary that other instances can consume. The protocol ensures consistency across platforms and instances, making context transfer reliable regardless of who produced the summary or who will read it.

---

## How to Use

1. Copy this template into your AI instance's instructions or paste it at the end of a session
2. Ask the instance to produce a summary following this format
3. Save the output to your shared memory layer (e.g., Google Drive /summaries/ folder)
4. Name the file: `[instance_name]_[YYYY-MM-DD].md`

---

## Template

```markdown
# Session Summary: [Instance Name]
## Date: [YYYY-MM-DD]
## Project: [project name]
## Duration: [approximate, e.g., "~2 hours"]

### What happened
[2-3 sentences describing the primary work accomplished this session. Be specific about deliverables produced, decisions made, or problems solved. Avoid vague summaries like "worked on the project."]

### Key decisions made
- [Decision 1: what was decided and why]
- [Decision 2: what was decided and why]
- [Add more as needed]

### Open items
- [Item 1: what still needs to happen, who owns it]
- [Item 2: what still needs to happen, who owns it]
- [Add more as needed]

### Context for other instances
- [Target instance name]: [What they need to know and why. Be specific about what changed that affects their work.]
- [Another target instance, if applicable]: [What they need to know]

### Corrections to shared memory
- [If any terminology, fact, or preference was corrected this session, note it here with which file in /memory/ should be updated. Example: "Update terminology.md: use 'multi-instance' not 'multi-agent' everywhere."]
- [If none: "No corrections."]
```

---

## Guidelines

**Be specific, not comprehensive.** The summary should capture what another instance needs to know to stay current, not everything that happened. If a 30-minute discussion led to one decision, capture the decision, not the discussion.

**Name target instances explicitly.** "Context for other instances" is the most important field for dispatch. If you know which instance needs the information, say so. If the information is relevant to all instances, say "All."

**Corrections are critical.** A correction that doesn't propagate will resurface as an error in another instance. Always capture corrections, even small ones.

**Summaries are not transcripts.** Two to three sentences for "What happened" is enough. The summary exists to transfer context, not to document the session.

---

*From the [Context Architecture](https://github.com/asanch3z/context-architecture) repo. Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
