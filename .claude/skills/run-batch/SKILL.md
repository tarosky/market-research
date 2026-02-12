---
name: run-batch
description: context.mdから全12分析を依存関係に基づいて並列バッチ実行する。全分析を一括実行したいとき、新しいビジネスの分析を最初から最後まで通したいときに使用する。
disable-model-invocation: true
allowed-tools: Read, Glob, Skill, Task, AskUserQuestion
---

# バッチ分析実行スキル

context.md を起点に、全12分析コマンドを依存関係に基づいて4フェーズで並列実行する。

## 前提条件

- `context.md` が存在すること。なければユーザーに `/init` を先に実行するよう案内して終了する
- `output/` ディレクトリが存在すること

## 実行前の確認

まず AskUserQuestion で以下を確認する:

**質問1: 実行範囲**
- header: "実行範囲"
- question: "既存の分析ファイルがある場合、どうしますか？"
- options:
  - "すべて再実行" — output/ 内の既存ファイルを上書きして全分析をやり直す
  - "未実行のみ" — 既存ファイルがある分析はスキップし、未実行の分析のみ実行する
- multiSelect: false

## フェーズ構成

依存関係に基づき、以下の4フェーズで実行する。
各フェーズ内の分析は **Task ツールで並列に** サブエージェント（subagent_type: "general-purpose"）を起動して実行する。

### Phase 1: 基礎分析（並列4タスク）

context.md のみに依存する独立した分析:

| タスク | コマンド | 出力ファイル |
|--------|---------|------------|
| 市場規模 | `/tam` | `output/01-tam.md` |
| トレンド | `/trends` | `output/04-trends.md` |
| 競合分析 | `/competitive` | `output/02-competitive.md` |
| ペルソナ | `/persona` | `output/03-persona.md` |

各サブエージェントへのプロンプト例:

```
このプロジェクトは市場調査ツールです。
context.md を読み込み、/tam コマンドを実行してください。
Skill ツールで Skill("tam") を呼び出してください。
```

**4タスクすべての完了を待ってから Phase 2 に進む。**

### Phase 2: クロスリファレンス分析（並列3タスク）

Phase 1 の出力を参照する分析:

| タスク | コマンド | 出力ファイル | 主な参照元 |
|--------|---------|------------|-----------|
| SWOT | `/swot` | `output/05-swot.md` | competitive, trends |
| 価格戦略 | `/pricing` | `output/06-pricing.md` | competitive, persona, tam |
| ジャーニー | `/journey` | `output/08-journey.md` | persona |

**3タスクすべての完了を待ってから Phase 3 に進む。**

### Phase 3: 統合型分析（並列4タスク）

Phase 1 + 2 の出力を参照する分析:

| タスク | コマンド | 出力ファイル | 主な参照元 |
|--------|---------|------------|-----------|
| 財務モデル | `/financial` | `output/09-financial.md` | tam, pricing |
| 海外展開 | `/expansion` | `output/11-expansion.md` | tam, competitive, trends |
| GTM戦略 | `/gtm` | `output/07-gtm.md` | 全出力ファイル |
| リスク評価 | `/risk` | `output/10-risk.md` | 全出力ファイル |

**4タスクすべての完了を待ってから Phase 4 に進む。**

### Phase 4: 統合戦略提言（単独）

全出力ファイルを統合する最終分析。これはメインコンテキストで直接 Skill("synthesis") を呼び出す（サブエージェントではなく）:

| タスク | コマンド | 出力ファイル |
|--------|---------|------------|
| 統合戦略 | `/synthesis` | `output/12-synthesis.md` |

## 実行中の進捗報告

各フェーズの開始時と完了時に、ユーザーに進捗を報告する:

```
Phase 1/4: 基礎分析を並列実行中... (tam, trends, competitive, persona)
Phase 1/4: 完了 ✓

Phase 2/4: クロスリファレンス分析を並列実行中... (swot, pricing, journey)
Phase 2/4: 完了 ✓

Phase 3/4: 統合型分析を並列実行中... (financial, expansion, gtm, risk)
Phase 3/4: 完了 ✓

Phase 4/4: 統合戦略提言を実行中... (synthesis)
Phase 4/4: 完了 ✓
```

## スキップ処理（「未実行のみ」選択時）

ユーザーが「未実行のみ」を選択した場合:
1. 各フェーズ開始前に、対象の出力ファイルが既に存在するか Glob で確認する
2. 存在するファイルの分析はスキップし、「XXX: スキップ（既存ファイルあり）」と報告する
3. フェーズ内の全タスクがスキップ対象なら、そのフェーズ自体をスキップする

## 完了時の報告

全フェーズ完了後、以下を報告する:
- 実行した分析の数
- スキップした分析の数（あれば）
- 生成されたファイル一覧
- 「/slides で分析結果をプレゼンテーションに変換できます」と案内する
