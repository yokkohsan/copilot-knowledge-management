# Copilot Knowledge Management

## Problem Statement

### 日本語（JP）

Microsoft Copilot（特に Teams 内 Copilot）を用いた長文の検討・設計・成果物作成では、
以下の構造的な問題が避けられません。

- チャットの会話上限により、思考や成果物が途中で分断される
- フォルダ URL を渡しても、Copilot は内部ファイルを列挙・参照できない
- ファイル URL を貼り付けたときの「添付扱い」挙動が環境によって異なる
- 長い Markdown やネストしたコードブロックが、会話継続の中で破綻しやすい
- Copilot は長期的な状態や前提を保持できない

本リポジトリは、これらの制約を前提として受け入れた上で、
**思考・判断・一次成果物を失わずに Copilot を運用するための方法とテンプレート**
を提供します。

---

### English (EN)

When using Microsoft Copilot (especially Teams Copilot) for long-form discussions,
design work, or artifact creation, the following structural limitations are unavoidable:

- Chat history has a hard limit, interrupting long-running reasoning
- Copilot cannot enumerate or traverse files via folder URLs
- Pasting file URLs triggers “attachment-like” behavior depending on the environment
- Long Markdown texts and nested code blocks tend to break across chat continuations
- Copilot does not maintain long-term state or assumptions across sessions

This repository provides **a practical and reproducible way to work with Copilot
without losing reasoning context or primary artifacts**, by explicitly designing
around these constraints.

---

## What this repository is

This repository is a **distribution package**.

It contains:
- Prompt templates (rescue / backup / merge / restore)
- A MANIFEST template for controlling Copilot’s reference scope
- Documentation explaining the design principles

This repository itself is **not** an operational workspace.

---

## Repository layout (GitHub)

```text
copilot-knowledge-management/
├ README.md
├ Project_MANIFEST.template.txt
├ LICENSE
├ prompts/
│  ├ ja/
│  └ en/
└ docs/
   ├ ja/
   └ en/
```

- `prompts/`  
  Prompt templates used for each knowledge-handling phase
- `Project_MANIFEST.template.txt`  
  Template for explicitly defining which files Copilot is allowed to read
- `docs/`  
  Additional explanations and usage notes (JP / EN)

---

## Working directory layout (OneDrive / SharePoint)

Actual Copilot operations are performed in a separate working directory,
typically located in OneDrive or SharePoint so that Teams Copilot can access the files.

Example:

```text
/Copilot_Knowledge
├ README.md
├ /ProjectX
│  ├ ProjectX_MANIFEST.txt
│  ├ merged.md
│  ├ ProjectX_YYYYMMDD_summary_N.md
│  └ history/
└ /ProjectY
   └ ...
```

This working directory does **not** need to mirror the GitHub repository layout.

---

## How to use

Copy the required prompt files and the MANIFEST template from this repository
into your own working directory (OneDrive / SharePoint), and operate Copilot there.

---

## Design principles

- Copilot is treated as a stateless reasoning engine, not a memory store
- The authoritative knowledge always lives on the human side (Markdown files)
- Reference scope must be explicitly declared, never inferred
- Operational stability is prioritized over convenience or automation

---

## Versions / Releases

This repository is versioned using GitHub Releases.

See the **Releases** page for:
- Incremental history from v0.1 to v1.x
- Version-specific intent and scope
- Distributed zip archives

---

## License

See `LICENSE`.
