# Market Research Assistant

AIを活用した市場調査・戦略分析ツール。
[Claude Code](https://docs.anthropic.com/en/docs/claude-code) のスラッシュコマンドで対話的にビジネス分析を実行し、Markdownレポートとして出力します。

## 必要なもの

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（インストール済みであること）
- [Node.js](https://nodejs.org/) 18以上（スライド生成機能を使う場合）

## セットアップ

```bash
git clone https://github.com/tarosky/market-research.git
cd market-research
npm install   # スライド生成用の依存パッケージをインストール
```

## 使い方

### 1. コンテキストの作成

最初に `/init` を実行して、分析対象のビジネス情報を収集します。

```
/init
```

対話形式でプロダクト概要・ターゲット顧客・ビジネスモデルなどをヒアリングし、`context.md` に保存します。
すべての分析コマンドはこの `context.md` を参照するため、最初に必ず実行してください。

### 2. 分析の実行

以下のコマンドを任意の順序で実行できます。各コマンドは `output/` 以下にナンバリング付きMarkdownを生成します。

| コマンド | 出力ファイル | 内容 |
|---------|------------|------|
| `/tam` | `output/01-tam.md` | 市場規模（TAM / SAM / SOM）分析 |
| `/competitive` | `output/02-competitive.md` | 競合分析 |
| `/persona` | `output/03-persona.md` | 顧客ペルソナ・セグメンテーション |
| `/trends` | `output/04-trends.md` | 業界トレンド分析 |
| `/swot` | `output/05-swot.md` | SWOT ＋ ポーターの5フォース分析 |
| `/pricing` | `output/06-pricing.md` | 価格戦略 |
| `/gtm` | `output/07-gtm.md` | Go-To-Market 戦略 |
| `/journey` | `output/08-journey.md` | カスタマージャーニーマップ |
| `/financial` | `output/09-financial.md` | 財務モデル・ユニットエコノミクス |
| `/risk` | `output/10-risk.md` | リスク評価・シナリオプランニング |
| `/expansion` | `output/11-expansion.md` | 市場参入・拡大戦略 |
| `/synthesis` | `output/12-synthesis.md` | 統合戦略提言 |

### 3. 推奨ワークフロー

```
/init          # まずビジネスコンテキストを収集
  ↓
/tam           # 市場規模を把握
  ↓
/competitive   # 競合環境を理解
  ↓
/persona       # 顧客像を明確化
  ↓
（必要な分析を追加実行）
  ↓
/synthesis     # 全分析を統合して戦略提言
```

各分析は既存の `output/` 内ファイルを参照して整合性を保つため、基礎分析（TAM → 競合 → ペルソナ）を先に済ませるとより精度の高い結果が得られます。

### 4. スライド生成

分析レポートをプレゼンテーション資料に変換できます。

```
/slides
```

対話形式で以下を選択します：

1. **ソースファイル** — `output/` 内の分析ファイルから選択
2. **聴衆** — CEO・投資家・チームメンバー・カンファレンス登壇、または自由入力
3. **出力形式** — HTMLのみ、または HTML + PDF

選択に応じてスライドの枚数・トーン・構成が自動調整され、`output/slides/` に出力されます。

```
output/slides/
├── 12-synthesis-slides.md     # Marp用マークダウン（編集可能）
├── 12-synthesis-slides.html   # ブラウザで閲覧
└── 12-synthesis-slides.pdf    # PDF（選択時のみ）
```

内部的には [Marp](https://marp.app/) を使用しています（`npm install` で自動インストール済み）。

## ディレクトリ構成

```
market-research/
├── .claude/
│   ├── CLAUDE.md              # プロジェクト設定・ルール
│   ├── commands/              # スラッシュコマンド定義（分析系）
│   │   ├── init.md
│   │   ├── tam.md
│   │   ├── competitive.md
│   │   └── ...
│   ├── skills/                # スキル定義（拡張機能）
│   │   └── slides/
│   │       └── SKILL.md       # /slides スライド生成スキル
│   ├── settings.json          # 共通パーミッション設定
│   └── settings.local.json    # 個人パーミッション設定（Git管理外）
├── context.md                 # ビジネスコンテキスト（/init で生成）
├── output/                    # 分析レポート出力先
│   ├── 01-tam.md
│   ├── 02-competitive.md
│   ├── ...
│   └── slides/                # プレゼンテーション出力先
├── package.json               # 依存パッケージ（Marp CLI）
└── README.md
```

## パーミッション設定

初回実行時、Web検索やファイル書き込みの許可を求められる場合があります。
スムーズに進めるには、`.claude/settings.local.json` に以下を追加してください：

```json
{
  "permissions": {
    "allow": [
      "Edit",
      "Write"
    ]
  }
}
```

Web検索・ファイル読み取り系のパーミッションはリポジトリの `.claude/settings.json` で共通設定済みです。

## 出力ルール

- 言語は日本語、専門用語は英語併記（例：「総市場規模（TAM: Total Addressable Market）」）
- 数値には根拠・出典・仮定を明記、推定値は「推定」と明示
- 可能な限り Web 検索で最新データを収集

## Acknowledgements

このプロジェクトは [@socialwithaayan](https://x.com/socialwithaayan) の[ポスト](https://x.com/socialwithaayan/status/2021233357514997824)にインスパイアされています。

## License

MIT
