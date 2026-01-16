# Copilot Knowledge Management 運用ガイド（日本語）

このドキュメントは、Copilot Knowledge Management を **実際に運用するための全体像・手順・命名規則** を 1 つにまとめたものです。

Copilot には複数の UI（Web、Teams、ローカルアプリ）が存在しますが、本運用は **特定の UI に依存せず、同じ前提を安定して再現できること** を目的としています。

---

## 全体思想

- Copilot は UI によって参照の優先順位や文脈解釈が変わります
- 人間側が、知識・判断・成果物の正本を管理します
- Copilot には、その都度 **参照してよい範囲だけ** を明示します
- 会話上限や UI の違いを前提として、破綻しない運用を行います

---

## 運用ディレクトリ構成（推奨例）

以下は、複数プロジェクトを継続的に運用している **実際の運用状態に近い一例** です。

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

- Copilot に渡す知識は **常に Project ディレクトリ単位**
- `_common/` は Copilot に渡さない共通物置き場（任意）

---

## ファイル命名規則

### 正本（常に 1 つ）

- `merged.md`
  - Project ディレクトリ直下に置く
  - 現時点で有効な正本

---

### summary（差分・スナップショット）

- `ProjectX_YYYYMMDD_summary_N.md`
  - YYYYMMDD：作成日
  - N：論理的な連番（大きいほど新しい）
  - Project 名を含めることで、移動・混在しても判別可能

---

### history に保存する merged

- `ProjectX_YYYYMMDD_merged.md`
  - YYYYMMDD：その merged を作成・更新した日
  - 過去の正本を履歴として保存するためのファイル

---

## MANIFEST の考え方

- MANIFEST は **Copilot に与える前提範囲の宣言**
- Copilot の UI や参照優先順位に依存しないための要です
- MANIFEST に書かれていない情報は、前提として扱わせません

---

### 基本的な使い方

- `_common/Project_MANIFEST.template.txt` をコピー
- Project 用に `ProjectX_MANIFEST.txt` として保存
- `[KNOWLEDGES]` に、現時点で有効な
  - `merged.md`
  - 最新の summary
  の URL を記載
- `[HISTORY]` には、過去の summary / merged を必要に応じて記載

---

## 運用フロー概要

本運用は、次の 4 ステップを状況に応じて使い分けます。

- rescue（非常時の救出）
- backup（通常保存）
- merge（正本化）
- restore（前提復元）

---

## ステップ 1：rescue（非常時の知識救出）

### 目的
- 会話上限に達した後でも、可能な限り正確に知識と成果物を救出する

### 使用するもの
- `knowledge_rescue_prompt.txt`
- チャットログのスクリーンショット等

### ルール
- 新しい内容を追加しない
- 推測・補完を行わない
- 不明な点は「不明」と明記する

---

## ステップ 2：backup（通常の知識保存）

### 目的
- 会話上限に達する前に、知識と一次成果物を完全保存する

### 使用するもの
- `knowledge_backup_prompt.txt`

### ルール
- 一次成果物は原文のまま保存する
- 知識バックアップ文書自体は成果物として扱わない

---

## ステップ 3：merge（正本の作成・更新）

### 目的
- 複数の summary / rescue から、最新の正本を作成・更新する

### 使用するもの
- `knowledge_merge_prompt.txt`
- `ProjectX_MANIFEST.txt` の `[KNOWLEDGES]`

### ルール
- `merged.md` を常に正本とする
- 古い `merged.md` は history に退避する
- 一次成果物のみを正本に含める

---

## ステップ 4：restore（新しいチャットでの再開）

### 目的
- 新しいチャットで、直前までの前提を再現する

### 使用するもの
- `knowledge_restore_prompt.txt`
- `ProjectX_MANIFEST.txt` の `[KNOWLEDGES]`

### ルール
- 新しい結論を出さない
- 前提の復元に専念する
- restore 完了後に、通常の議論を再開する

---

## どこまで Copilot に見せるか

- 通常の restore / merge で参照させるもの
  - `merged.md`
  - 最新の summary
- `history/` は人間側の履歴管理用であり、通常は Copilot に渡しません

---

## 最後に

この運用で最も重要なのは、

> **Copilot の UI が変わっても、同じ前提を与えられること**

です。

それを実現するために、
- MANIFEST
- 命名規則
- 正本（merged.md）

を人間側で管理します。
