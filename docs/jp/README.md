# Copilot Knowledge Management 運用ガイド（日本語）

このドキュメントは、Copilot Knowledge Management を **実際に運用するための全体像・手順・命名規則** を 1 つにまとめたものです。

Copilot には複数の UI（Web、Teams、ローカルアプリ）が存在しますが、本運用は **特定の UI に依存せず、同じ前提と成果物を安定して再現できること** を目的としています。

---

## 全体思想

- Copilot は UI によって、参照される情報の優先順位や文脈解釈が変わります
- 人間側が、前提知識・判断・成果物の正本を管理します
- Copilot には、その都度 **参照してよい範囲だけ** を明示します
- 会話上限や UI の違いを前提として、破綻しない運用を行います

---

## 運用ディレクトリ構成（推奨例）

以下は、複数プロジェクトを継続的に運用している **実際の運用状態に近い一例** です。  
前提知識だけでなく、成果物が段階的に増えていくことを想定しています。

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

- Copilot に渡す知識は **常に Project ディレクトリ単位** とします
- `_common/` は Copilot に渡さない共通物置き場（任意）です

---

## ファイル命名規則

### 正本（常に 1 つ）

- `merged.md`
  - Project ディレクトリ直下に配置します
  - 現時点で有効な **前提知識と成果物の正本** です

---

### summary（差分・スナップショット）

- `ProjectX_YYYYMMDD_summary_N.md`
  - YYYYMMDD：作成日
  - N：論理的な連番（大きいほど新しい）
  - 主に **前提知識の変化や検討結果** を保存します
  - Project 名を含めることで、移動・混在しても判別可能です

---

### history に保存する merged

- `ProjectX_YYYYMMDD_merged.md`
  - YYYYMMDD：その merged を作成・更新した日
  - 過去の **前提知識＋成果物の正本** を履歴として保存します

---

## なぜ Notebook / フォルダ参照を使わないか

Copilot Notebook やフォルダ参照の仕組みも検討しましたが、実運用においては **知識継承の完全な解決策にはなりませんでした**。

具体的には、以下の問題が確認されています。

- 内容が要約され、前提や判断の粒度が失われる
- 成果物（文章・設計・プログラムコード）が完全な形で残らない
- なぜその結論に至ったのかを、後から追えなくなる

本運用では、ファイルの共有 URL をプロンプトに貼り付け、Copilot に **添付相当として認識させる挙動** を前提とすることで、原文のまま知識と成果物を参照・復元できるようにしています。

---

## MANIFEST の考え方

- MANIFEST は **Copilot に与える参照範囲の宣言** です
- Copilot の UI や参照優先順位に依存しないための要となります
- MANIFEST に書かれていない情報は、前提として扱わせません

### MANIFEST を更新するタイミング

MANIFEST は「ファイル管理台帳」ではなく、Copilot に貼り付ける **参照用 URL セット** です。

以下のタイミングで更新します。

- merge により `merged.md` を更新したとき
- restore で参照させたい summary を変更したとき
- 古い summary を参照対象から外したいとき

通常は、

- `[KNOWLEDGES]` に最新の `merged.md` と必要最小限の summary
- `[HISTORY]` に過去の参照履歴

のみを記載します。

---

## 運用フロー概要

本運用は、次の 4 ステップを状況に応じて使い分けます。

- rescue（救出）
- backup（保存）
- merge（正本化）
- restore（前提の復元）

---

## ステップ 1：rescue（非常時の知識・成果物救出）

### 実施タイミング
- チャットがすでに上限に達している、または到達直後

### 目的
- すでにチャット上限に達している内容から、  
  前提知識と成果物を可能な限り正確に救出します

### 使用するもの
- `knowledge_rescue_prompt.txt`
- チャット内容のスクリーンショット

### ルール
- 新しい内容を追加しません
- 推測・補完を行いません
- 不明な点は「不明」と明記します
- 成果物は **原文のまま** 記録します

---

## ステップ 2：backup（通常の知識・成果物保存）

### 実施タイミング
- 会話が長期化し、前提知識や成果物が積み上がってきたとき
- チャット上限に達する前

### 目的
- 会話上限に達する前に、**前提知識と成果物の最新版を確実に保存** します

### 使用するもの
- `knowledge_backup_prompt.txt`

### 出力内容
- 前提知識（判断・結論・未解決事項など）
- **成果物セクション**
  - 文章原稿
  - 設計書
  - プログラムコード など

### ルール
- 成果物は要約せず、原文のまま出力します
- Markdown やコードブロックを含む場合は、**出力全体を十分に長いバッククオートで囲みます**
- 知識バックアップ文書そのものは成果物として扱いません

---

## ステップ 3：merge（正本の作成・更新）

### 実施タイミング
- backup や rescue により、新しい成果物が得られたとき
- 正本（`merged.md`）を更新したいとき

### 目的
- 複数の summary / rescue から、**前提知識と成果物を含む最新の正本** を作成・更新します

### 使用するもの
- `knowledge_merge_prompt.txt`
- `ProjectX_MANIFEST.txt` の `[KNOWLEDGES]`

### ルール
- 参照した前提知識・成果物の **確認ログ** を先に出力します
- 確認後に、コピー用の `merged.md` 本文を出力します
- 一次成果物のみを正本に含めます
- 古い `merged.md` は history に退避します

---

## ステップ 4：restore（前提のみの復元）

### 実施タイミング
- 新しいチャットで、過去の議論を再開したいとき

### 目的
- 新しいチャットで、**議論を再開するための前提のみを復元** します

### 使用するもの
- `knowledge_restore_prompt.txt`
- `ProjectX_MANIFEST.txt` の `[KNOWLEDGES]`

### ルール
- 新しい結論を出しません
- 前提の復元に専念します
- 成果物は再掲せず、**メタ情報（存在・種類・位置づけ）のみを提示** します
- 復元完了後に、通常の議論を再開します

---

## どこまで Copilot に見せるか

- 通常の merge / restore で参照させるもの
  - `merged.md`
  - 最新の summary
- `history/` は人間側の履歴管理用であり、  
  通常は Copilot に渡しません

---

## 最後に

この運用で最も重要なのは、

> **前提知識を安全に復元しつつ、成果物を肥大化させないこと**

です。

restore はあくまで再開のための前提提示に限定し、成果物の保存・正本化は backup / merge に委ねます。
