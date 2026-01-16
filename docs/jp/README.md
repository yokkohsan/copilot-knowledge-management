# Copilot Knowledge Management 運用ガイド（日本語）

このドキュメントは、Copilot Knowledge Management を  
**実際に運用するための全体像・手順・命名規則**を 1 つにまとめたものです。

Copilot には複数の UI（Web、Teams、ローカルアプリ）が存在しますが、  
本運用は **特定の UI に依存せず、同じ一次成果物を安定して再現・再利用できること**  
を目的としています。

---

## 全体思想（v1.0）

- Copilot は UI によって、参照される情報の優先順位や文脈解釈が変わります
- 人間側が **一次成果物を正本として管理**します
- 前提知識・判断・検討経緯は、成果物を理解・再開するための補助情報として扱います
- Copilot には、その都度 **参照してよい範囲だけ** を明示します
- 会話上限や UI の違いを前提として、破綻しない運用を行います

---

## 運用ディレクトリ構成（推奨例）

以下は、複数プロジェクトを継続的に運用している  
**実際の運用状態に近い一例**です。  
一次成果物が段階的に成長し、それを正として管理することを前提としています。

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

- Copilot に渡す参照範囲は **常に Project ディレクトリ単位**
- `_common/` は Copilot に渡さない共通物置き場（任意）

---

## ファイル命名規則

### 正本（一次成果物を含む）

- `merged.md`
  - Project ディレクトリ直下に配置します
  - **一次成果物を含む唯一の正本**です
  - 前提知識・判断は、成果物を理解するための補助情報として含まれます

---

### summary（補助情報・差分）

- `ProjectX_YYYYMMDD_summary_N.md`
  - YYYYMMDD：作成日
  - N：論理的な連番（大きいほど新しい）
  - 前提知識・判断・検討経緯などの補助情報を保存します
  - 単体で正本にはなりません

---

### history に保存する merged

- `ProjectX_YYYYMMDD_merged.md`
  - YYYYMMDD：その merged を作成・更新した日
  - 過去の一次成果物＋補助情報の正本を履歴として保存します

---

## MANIFEST の考え方（v1.0）

- MANIFEST は **Copilot に与える参照範囲の宣言**です
- Copilot に「どの成果物が正か」を誤解させないための要です
- MANIFEST に書かれていないファイルは、前提として扱わせません

### 基本的な使い方

- `_common/Project_MANIFEST.template.txt` をコピーします
- Project 用に `ProjectX_MANIFEST.txt` として保存します
- `[KNOWLEDGES]` には、原則として
  - `merged.md`
  - 必要最小限の summary
  の URL のみを記載します
- `[HISTORY]` は履歴管理用で、通常の restore / merge では参照しません

---

## 運用フロー概要（検証済み）

本運用は、以下のループが **実運用で検証済み**です。

- rescue → rescue → backup → merge → restore
- restore 後に議論を継続し、再度 backup → merge → restore
- 上記を繰り返しても、正本や履歴が破綻しないことを確認しています

---

## ステップ 1：rescue（非常時の一次成果物救出）

### 目的
- すでにチャット上限に達している内容から、  
  **一次成果物と必要最小限の補助情報**を救出します

### 使用するもの
- `knowledge_rescue_prompt.txt`
- チャット内容のスクリーンショット

### ルール
- 新しい内容を追加しません
- 推測・補完を行いません
- 一次成果物は **原文のまま** 記録します
- 補助情報は、成果物を理解するために必要な範囲に限定します

---

## ステップ 2：backup（一次成果物の保存）

### 目的
- 会話上限に達する前に、  
  **一次成果物の最新版を確実に保存**します

### 使用するもの
- `knowledge_backup_prompt.txt`

### 出力内容
- 一次成果物（正本候補）
- 成果物を理解するために必要な最小限の補助情報

### ルール
- 一次成果物は要約せず、原文のまま出力します
- Markdown やコードブロックを含む場合は、  
  **出力全体を十分に長いバッククオートで囲みます**
- 補助情報は成果物の理解に必要な範囲に限定します

---

## ステップ 3：merge（正本の確定）

### 目的
- 複数の backup / rescue から、  
  **一次成果物を正として統合した正本**を確定します

### 使用するもの
- `knowledge_merge_prompt.txt`
- `ProjectX_MANIFEST.txt` の `[KNOWLEDGES]`

### ルール
- 参照した成果物・補助情報の **確認ログ**を先に出力します
- 確認後に、コピー用の `merged.md` 本文を出力します
- 一次成果物のみを正として扱います
- 古い `merged.md` は history に退避します

---

## ステップ 4：restore（一次成果物を起点とした再開）

### 目的
- 新しいチャットで、  
  **一次成果物を正本として再開**します

### 使用するもの
- `knowledge_restore_prompt.txt`
- `ProjectX_MANIFEST.txt` の `[KNOWLEDGES]`

### ルール
- 参照した成果物・補助情報の **確認ログ**を出力します
- 復元された一次成果物を明示します
- 新しい結論は出さず、復元に専念します
- 復元完了後に、通常の議論を再開します

---

## どこまで Copilot に見せるか

- 通常の merge / restore で参照させるもの
  - `merged.md`（一次成果物を含む正本）
  - 必要最小限の summary
- `history/` は人間側の履歴管理用であり、  
  通常は Copilot に渡しません

---

## 最後に（v1.0）

この運用で最も重要なのは、

> **「一次成果物を正として守り続けられること」**

です。

前提知識や判断は変化しても、  
積み上げた成果物を失わず、  
同じ成果物を起点に議論を再開できる状態を維持します。
