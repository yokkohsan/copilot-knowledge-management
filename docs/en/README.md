# Copilot Knowledge Management – Operation Guide (English)

This document describes the **complete operational model, steps, and naming rules** for running Copilot Knowledge Management in practice.

Microsoft Copilot has multiple UIs (Web, Teams, desktop apps), but this workflow is designed so that **the same assumptions and artifacts can be restored and reused reliably, regardless of the UI**.

---

## Core Principles

- Copilot may change its reference priority and interpretation depending on the UI
- Humans manage the authoritative content: assumptions, decisions, and artifacts
- Copilot is always given **an explicit reference scope**
- The workflow is designed to remain stable despite chat limits or UI differences

---

## Recommended Directory Structure

The following example reflects **real-world operation**, where assumptions and artifacts grow over time.

```text
Copilot_Knowledge/
├ README.md
├ _common/
│  ├ prompts/
│  │  ├ knowledge_rescue_prompt.txt
│  │  ├ knowledge_backup_prompt.txt
│  │  ├ knowledge_merge_prompt.txt
│  │  └ knowledge_restore_prompt.txt
│  └ Project_MANIFEST.template.txt
├ ProjectX/
│  ├ merged.md
│  ├ ProjectX_20260120_summary_4.md
│  ├ ProjectX_20260122_summary_5.md
│  ├ ProjectX_MANIFEST.txt
│  └ history/
│     ├ ProjectX_20260110_summary_1.md
│     ├ ProjectX_20260112_summary_2.md
│     ├ ProjectX_20260115_merged.md
│     ├ ProjectX_20260118_summary_3.md
│     └ ProjectX_20260120_merged.md
└ ProjectY/
   └ ...
```

- Copilot is always given references **per project directory**
- `_common/` is optional and is **not passed to Copilot**

---

## File Naming Rules

### Authoritative File (Single Source of Truth)

- `merged.md`
  - Located directly under each project directory
  - The authoritative file that includes assumptions and artifacts

---

### summary Files (Supporting Snapshots)

- `ProjectX_YYYYMMDD_summary_N.md`
  - YYYYMMDD: creation date
  - N: logical sequence number (larger means newer)
  - Stores changes in assumptions and discussion context
  - Not authoritative on its own

---

### Historical merged Files

- `ProjectX_YYYYMMDD_merged.md`
  - YYYYMMDD: date when the merged file was created or updated
  - Stores past authoritative states for reference only

---

## Why We Do Not Use Notebooks or Folder References

Copilot Notebooks and folder-based references were evaluated, but they did not work well for long-term knowledge continuity in real operation.

The following issues were observed:

- Content tends to be summarized, losing important assumptions and decisions
- Artifacts (documents, designs, code) are not preserved in full
- It becomes difficult to trace why a specific conclusion was reached

This workflow instead relies on **pasting file URLs into the prompt**, so that Copilot recognizes them as attachments.  
This allows assumptions and artifacts to be restored **in their original form**.

---

## MANIFEST Concept

- The MANIFEST explicitly declares **what Copilot is allowed to read**
- It prevents Copilot from misinterpreting which files are authoritative
- Any file not listed in the MANIFEST must not be treated as an assumption

### When to Update the MANIFEST

The MANIFEST is **not a file inventory**.  
It is a set of file URLs that are pasted into Copilot.

Update the MANIFEST in the following cases:

- When `merged.md` is updated by a merge
- When the summary files referenced during restore change
- When old summaries should no longer be referenced

In normal operation:

- `[ACTIVE_KNOWLEDGES]` contains the latest `merged.md` and the minimum required summaries
- `[HISTORY]` contains past references for record-keeping only

During normal merge / restore operations, paste only the URLs in `[ACTIVE_KNOWLEDGES]` into Copilot. Do not pass `[HISTORY]` to Copilot unless it is explicitly needed.

---

## Operational Flow Overview

This workflow consists of the following four steps, executed depending on the situation:

- rescue
- backup
- merge
- restore

---

## Step 1: rescue (Emergency Recovery)

### When to run
- When the chat has already reached the limit, or immediately after

### Purpose
- Recover assumptions and artifacts from a conversation that can no longer continue due to chat limits

### What to use
- `knowledge_rescue_prompt.txt`
- Screenshots of the chat content

### Rules
- Do not add new content
- Do not guess or generalize
- Clearly mark unknown information
- Record artifacts **verbatim**

---

## Step 2: backup (Normal Preservation)

### When to run
- When discussions become long and assumptions or artifacts start to accumulate
- Before reaching the chat limit

### Purpose
- Preserve the **latest assumptions and artifacts** safely

### What to use
- `knowledge_backup_prompt.txt`

### Output includes
- Assumptions (decisions, constraints, open issues)
- **Artifact section**
  - Documents
  - Designs
  - Program code

### Rules
- Do not summarize artifacts
- Preserve original content exactly
- If Markdown or code blocks are included, **wrap the entire output with sufficiently long backticks**
- Backup documents themselves are not treated as artifacts

---

## Step 3: merge (Update the Authoritative File)

### When to run
- After new artifacts are created via backup or rescue
- When the authoritative file should be updated

### Purpose
- Create or update the authoritative `merged.md` from backups or rescued content

### What to use
- `knowledge_merge_prompt.txt`
- `[ACTIVE_KNOWLEDGES]` section of `ProjectX_MANIFEST.txt`

### Rules
- Output a verification log first
- Then output the copy-ready `merged.md`
- Treat artifacts as authoritative
- Move older `merged.md` files into `history/`

---

## Step 4: restore (Restore Assumptions Only)

### When to run
- When starting a new chat to continue past discussions

### Purpose
- Restore **assumptions only** to restart discussion safely

### What to use
- `knowledge_restore_prompt.txt`
- `[ACTIVE_KNOWLEDGES]` section of `ProjectX_MANIFEST.txt`

### Rules
- Do not produce new conclusions
- Focus only on restoring assumptions
- Do not reprint artifacts
- Show **artifact metadata only** (existence, type, role)
- After restoration, resume normal discussion

---

## What Copilot Is Allowed to See

- During merge and restore:
  - `merged.md`
  - The minimum required summary files
- `history/` is for human reference only and is not passed to Copilot

---

## Final Notes

The most important goal of this workflow is:

> **To restore assumptions safely without causing artifact duplication or growth**

By separating backup/merge (preservation) from restore (restart), long-term operation remains stable and predictable.
