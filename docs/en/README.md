(Validation status: not started)

# Copilot Knowledge Management – Operation Guide (English)

This document explains the **full operational model, steps, and naming rules**
for running Copilot Knowledge Management in practice.

Microsoft Copilot has multiple UIs (Web, Teams, desktop apps),
but this workflow is designed so that **the same assumptions and artifacts
can be restored and reused reliably, regardless of the UI**.

---

## Core Principles

- Copilot may change its reference priority and interpretation depending on the UI
- Humans manage the authoritative content: assumptions, decisions, and artifacts
- Copilot is always given **an explicit reference scope**
- The workflow is designed to remain stable despite chat limits or UI differences

---

## Recommended Directory Structure

The following is an example close to **real-world operation**,
where assumptions and primary artifacts grow over time.

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
  - The **only authoritative file** that includes primary artifacts
  - Assumptions and decisions are included only as supporting context

---

### summary Files (Snapshots / Supporting Context)

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

## MANIFEST Concept

- The MANIFEST explicitly declares **what Copilot is allowed to read**
- It prevents Copilot from misinterpreting which artifact is authoritative
- Any file not listed in the MANIFEST must not be treated as an assumption

### Basic Usage

- Copy `_common/Project_MANIFEST.template.txt`
- Save it as `ProjectX_MANIFEST.txt` for each project
- In `[KNOWLEDGES]`, list only:
  - `merged.md`
  - The minimum required summary files
- `[HISTORY]` is for record-keeping and is not used in normal restore or merge

---

## Operational Flow (Verified)

The following loop has been **verified in real operation**:

- rescue → rescue → backup → merge → restore
- After restore, continue discussion and repeat backup → merge → restore
- This loop does not cause unbounded growth or structural breakage

---

## Step 1: rescue (Emergency Recovery)

### Purpose
- Recover assumptions and **primary artifacts** after chat limits are reached

### What to Use
- `knowledge_rescue_prompt.txt`
- Screenshots of the chat content

### Rules
- Do not add new content
- Do not guess or generalize
- Record primary artifacts **verbatim**
- Limit supporting context to what is necessary

---

## Step 2: backup (Normal Preservation)

### Purpose
- Preserve the **latest primary artifacts** before reaching chat limits

### What to Use
- `knowledge_backup_prompt.txt`

### Output Includes
- Primary artifacts (authoritative candidates)
- Minimum supporting assumptions needed to understand them

### Rules
- Do not summarize or edit primary artifacts
- Wrap the entire output with sufficiently long backticks
  if Markdown or code blocks are included
- Backup documents themselves are not treated as artifacts

---

## Step 3: merge (Finalize Authoritative State)

### Purpose
- Create or update the authoritative state
  by integrating multiple backups or rescues

### What to Use
- `knowledge_merge_prompt.txt`
- `[KNOWLEDGES]` section of `ProjectX_MANIFEST.txt`

### Rules
- Output a verification log first
- Then output the copy-ready `merged.md`
- Only primary artifacts are treated as authoritative
- Move older `merged.md` files into `history/`

---

## Step 4: restore (Assumption Restoration Only)

### Purpose
- Restore **assumptions only**, in order to restart discussion safely

### What to Use
- `knowledge_restore_prompt.txt`
- `[KNOWLEDGES]` section of `ProjectX_MANIFEST.txt`

### Rules
- Do not produce new conclusions
- Focus only on restoring assumptions
- Do not reprint artifacts
- Show **artifact metadata only** (existence, type, role)
- After restoration, resume normal discussion freely

---

## What Copilot Is Allowed to See

- During merge and restore:
  - `merged.md`
  - Minimum required summary files
- `history/` is for humans only and is not passed to Copilot

---

## Final Notes

The most important goal of this workflow is:

> **To restore assumptions safely without causing artifact duplication or growth**

Primary artifacts are preserved through backup and merge,
while restore is strictly limited to preparing a safe restart point.

This separation keeps long-term operation stable and predictable.
