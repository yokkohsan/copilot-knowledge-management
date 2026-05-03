# Copilot Knowledge Management

## Problem Statement

### 日本語（JP）

Microsoft Copilot（Web 版、Teams 内、ローカルアプリ内）を用いた長文の検討・設計・成果物作成では、以下の構造的な問題が避けられません。

- チャットや UI ごとに、参照される情報の優先順位や文脈解釈が異なる
- 会話履歴や作業文脈には上限があり、思考や成果物が途中で分断される
- フォルダ URL を渡しても、意図したとおりにファイルが参照される保証がない
- ファイル URL を貼り付けたときの挙動が環境や UI によって異なる
- 長い Markdown やネストしたコードブロックが、会話継続の中で壊れやすい
- Copilot は長期的な前提や状態を保証付きで保持しない

本リポジトリは、これらの制約を前提として受け入れた上で、**Copilot の UI や参照優先順位に依存せず、思考・判断・一次成果物を安定して再開できる運用方法**を提供します。

---

### English (EN)

When using Microsoft Copilot (Web, Teams, or in desktop apps) for long discussions, design work, or creating documents, the following problems are hard to avoid:

- The priority of referenced data and context changes depending on the Copilot UI
- Chat history and working context have limits, so long work gets interrupted
- Passing a folder URL does not guarantee that files are read as intended
- Pasting a file URL can behave differently depending on the UI or setup
- Long Markdown text and nested code blocks can break when chats continue
- Copilot does not reliably keep long-term assumptions across sessions

This repository provides **a practical way to work with Copilot without depending on a specific UI or reference priority**, by explicitly managing what context and knowledge are given to Copilot.

---

## What this repository is

This repository is a **distribution package**.

It provides:
- Prompt templates (rescue / backup / merge / restore)
- A MANIFEST template that lists active knowledge URLs under `[ACTIVE_KNOWLEDGES]`
- Documents that explain the operational ideas and rules

This repository itself is **not** a working directory.

This repository is mainly for Microsoft 365 Copilot knowledge continuity. It is not a replacement for GitHub Copilot repository instructions such as `.github/copilot-instructions.md`, `AGENTS.md`, or prompt files.

本リポジトリは、主に Microsoft 365 Copilot での知識継承を目的としています。GitHub Copilot のリポジトリ指示ファイル（`.github/copilot-instructions.md`、`AGENTS.md`、prompt files など）を置き換えるものではありません。

---

## About Copilot environments

This workflow is designed to work reliably in Microsoft 365 environments, especially when using Teams Copilot where cross-source access (Teams chats, SharePoint, organization-managed OneDrive, etc.) is required.

In some scenarios, using Copilot CLI or a local agent may be more suitable, for example:

- Complex code design or large-scale refactoring
- Changes that span an entire repository
- Tasks that require running tests or static analysis
- Long-term development of local artifacts in a structured project

In such cases, CLI-based workflows can complement this model. However, the core principles of artifact-first management and explicit reference control remain the same.

---

## Repository layout (GitHub)

```text
copilot-knowledge-management/
├ README.md
├ Project_MANIFEST.template.txt
├ LICENSE
├ prompts/
│  ├ jp/
│  └ en/
└ docs/
   ├ jp/
   └ en/
```

---

## Working directory layout (OneDrive / SharePoint)

Actual Copilot work is done in a **separate working directory**, usually placed in OneDrive or SharePoint so that Copilot can access the files regardless of the UI being used.

The structure below shows a **minimal example** that is sufficient for operation:

```text
Copilot_Knowledge/
├ README.md
├ ProjectX/
│  ├ merged.md
│  ├ ProjectX_YYYYMMDD_summary_N.md
│  ├ ProjectX_MANIFEST.txt
│  └ history/
│     ├ ProjectX_YYYYMMDD_summary_N.md
│     └ ProjectX_YYYYMMDD_merged.md
└ ProjectY/
   └ ...
```

More advanced or convenient layouts are described in the documentation under `docs/`.

---

## How to use

Copy the required prompt files and the MANIFEST template from this repository into your own working directory (OneDrive / SharePoint), and work there with Copilot.

In actual use, paste only the `[ACTIVE_KNOWLEDGES]` URLs from the MANIFEST into Copilot, and let Copilot report any URL it cannot reference.

---

## Design principles

- Copilot is treated as a tool that reads what it is explicitly given
- Knowledge and decisions are managed on the human side
- Reference scope must always be declared, never inferred
- The workflow should remain stable across different Copilot UIs

---

## Versions / Releases

This repository uses GitHub Releases.

The Releases page shows:
- The history from v0.1 to later versions
- What each version focused on
- Downloadable zip files

---

## License

See `LICENSE`.
