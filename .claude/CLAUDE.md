# Market Research Assistant

このプロジェクトは、AIを活用した市場調査・戦略分析ツールです。
Claude Codeのスラッシュコマンドを使い、対話的にビジネス分析を実行します。

## 基本ルール

### コンテキストの参照

- **すべての分析コマンド実行前に `context.md` を必ず読み込むこと**
- `context.md` が存在しない場合は、まず `/init` を実行するようユーザーに案内する
- `context.md` の情報が分析に不十分な場合は、ユーザーに追加質問してから進める

### 出力ルール

- 成果物は `output/` ディレクトリに `01-tam.md`, `02-competitive.md` のようにナンバリング付きMarkdownで保存する
- 言語は日本語。専門用語は英語を括弧書きで併記（例：「総市場規模（TAM: Total Addressable Market）」）
- 数値には必ず根拠・出典・仮定を明記する。推定値は「推定」と明示する
- 可能な限り web search で最新のデータを収集し、情報の鮮度を示す

### 分析間の連携

- 各分析コマンドは、`output/` 内の既存ファイルを参照して整合性を保つ
- 前の分析と矛盾する数値や仮定が出た場合は、ユーザーに指摘して判断を仰ぐ
- `/synthesis`（統合戦略）は `output/` 内の全ファイルを読み込んで実行する

### 分析の質を担保するために

- フレームワークの機械的な適用ではなく、ユーザーのビジネスに固有のインサイトを出すことを重視する
- 「一般的にはこうだが、あなたのケースでは…」という形で具体性を持たせる
- 確信度が低い分析には正直にその旨を記載し、追加調査の方向性を示す
- 必要に応じてユーザーに業界固有の情報や一次データの提供を求める

## コマンド一覧

| コマンド | 出力ファイル | 内容 |
|---------|------------|------|
| `/init` | `context.md` | ビジネスコンテキストの収集 |
| `/tam` | `output/01-tam.md` | 市場規模・TAM分析 |
| `/competitive` | `output/02-competitive.md` | 競合分析 |
| `/persona` | `output/03-persona.md` | 顧客ペルソナ・セグメンテーション |
| `/trends` | `output/04-trends.md` | 業界トレンド分析 |
| `/swot` | `output/05-swot.md` | SWOT＋ポーターの5F分析 |
| `/pricing` | `output/06-pricing.md` | 価格戦略 |
| `/gtm` | `output/07-gtm.md` | Go-To-Market戦略 |
| `/journey` | `output/08-journey.md` | カスタマージャーニー |
| `/financial` | `output/09-financial.md` | 財務モデル・ユニットエコノミクス |
| `/risk` | `output/10-risk.md` | リスク評価・シナリオプランニング |
| `/expansion` | `output/11-expansion.md` | 市場参入・拡大戦略 |
| `/synthesis` | `output/12-synthesis.md` | 統合戦略提言 |
| `/run-batch` | 上記すべて | 全12分析を依存関係に基づき並列バッチ実行 |
| `/slides` | `output/slides/` | 分析ファイルをMarpプレゼンテーションに変換 |

## 推奨ワークフロー

1. `/init` でコンテキスト収集
2. `/run-batch` で全分析を一括実行（または個別に実行）
3. 結果を見ながら必要な分析を再実行
4. `/slides` でプレゼンテーション資料を生成
