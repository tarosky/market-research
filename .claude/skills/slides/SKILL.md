---
name: slides
description: 分析マークダウンファイルからMarpプレゼンテーションスライドを生成する。output/内の分析ファイルをスライドに変換したいとき、プレゼン資料を作りたいときに使用する。
disable-model-invocation: true
allowed-tools: Read, Write, Glob, Bash(npx marp *), Bash(npx @marp-team/marp-cli *), WebSearch, AskUserQuestion
---

# Marp スライド生成スキル

分析マークダウンファイルを読み込み、Marp形式のプレゼンテーション用マークダウンを生成し、HTMLスライドに変換する。

## 手順

### 1. ユーザーへの質問（AskUserQuestion を使用）

`output/` ディレクトリ内の `.md` ファイルを Glob で一覧取得し、以下をまとめて AskUserQuestion で質問する:

**質問1: ソースファイル**
- header: "ソース"
- question: "どのファイルをスライドにしますか？"
- options: `output/` 内の `.md` ファイル一覧から動的に生成する（最大4つ。5つ以上ある場合は最近の4つを表示し、ユーザーが "Other" で指定できるようにする）
- multiSelect: false

**質問2: 聴衆**
- header: "聴衆"
- question: "誰向けのプレゼンですか？"
- options:
  - "CEO / 経営層" — 簡潔・結論先行、10〜15枚
  - "投資家" — 数字重視・ストーリー、15〜20枚
  - "チームメンバー" — 具体的・アクション志向、15〜25枚
  - "カンファレンス登壇" — ストーリー・インサイト重視、15〜20枚
- multiSelect: false

**質問3: 出力形式**
- header: "出力形式"
- question: "出力形式を選んでください"
- options:
  - "HTMLのみ" — ブラウザで閲覧。軽量で高速
  - "HTML + PDF" — PDF共有・印刷にも対応
- multiSelect: false

3つの質問を **1回の AskUserQuestion 呼び出し** でまとめて聞くこと。

### 2. ソースファイルの読み込み

- ユーザーが選択したファイルを読み込む
- `context.md` も読み込み、ビジネスコンテキストを把握する

### 3. 聴衆に合わせたスライド構成の設計

ユーザーが選択した聴衆に基づいて、スライドの構成・トーン・詳細度を調整する:

| 聴衆 | スライド枚数 | トーン | 重点 |
|------|------------|--------|------|
| CEO / 経営層 | 10〜15枚 | 簡潔・結論先行 | 意思決定に必要な情報のみ |
| 投資家 | 15〜20枚 | 数字重視・ストーリー | 市場規模・成長性・差別化 |
| チームメンバー | 15〜25枚 | 具体的・アクション志向 | 実行計画・担当・期限 |
| カンファレンス登壇 | 15〜20枚 | ストーリー・インサイト重視 | 学び・ユニークな視点 |
| その他（ユーザー指定） | 12〜18枚 | バランス型 | 全体像 + 重要ポイント |

### 4. Marpマークダウンの生成

以下のフォーマットに従ってMarp用マークダウンを生成する:

```markdown
---
marp: true
theme: default
paginate: true
header: "プレゼンタイトル"
footer: "© 2026"
style: |
  section {
    font-family: 'Helvetica Neue', Arial, 'Hiragino Kaku Gothic ProN', sans-serif;
  }
  h1 {
    color: #2d3748;
  }
  h2 {
    color: #4a5568;
  }
  table {
    font-size: 0.7em;
  }
---

# タイトルスライド

サブタイトル
日付

---

## スライド内容

- ポイント1
- ポイント2

---
```

#### Marpスライド作成のルール

1. **`---` でスライドを区切る**（frontmatterの後、各スライド間）
2. **1スライド1メッセージ**: 情報を詰め込みすぎない
3. **テーブルはコンパクトに**: 列数を3〜4に抑える。元のテーブルが大きい場合は分割または要約する
4. **太字で要点を強調**: 各スライドの最も重要なポイントを太字にする
5. **数字は大きく見せる**: 重要な数字は見出しレベルで表示する
6. **元の分析の「結論」を抽出**: 詳細な根拠はスライドに含めず、結論と要点のみ
7. **ストーリーラインを作る**: スライド全体を通じて一貫したナラティブを構成する

#### スライド構成の基本テンプレート

1. **タイトルスライド**: プロダクト名、サブタイトル、日付
2. **アジェンダ**: 今日話すこと（3〜5項目）
3. **現状サマリー**: ビジネスの現在地（1〜2枚）
4. **本編**: 分析の核心部分（聴衆に応じて調整）
5. **推奨アクション**: 次に何をすべきか（1〜2枚）
6. **まとめ / Q&A**: クロージング

### 5. ファイルの保存

- 元ファイル名からスライド用ファイル名を生成する
  - 例: `12-synthesis.md` → `output/slides/12-synthesis-slides.md`
- Marpマークダウンを `output/slides/` に保存する

### 6. HTML / PDFへの変換

保存したMarpマークダウンをHTMLスライドに変換する:

```bash
npx @marp-team/marp-cli output/slides/12-synthesis-slides.md --html -o output/slides/12-synthesis-slides.html
```

ユーザーが「HTML + PDF」を選択した場合は、PDFも追加で出力する:

```bash
npx @marp-team/marp-cli output/slides/12-synthesis-slides.md --pdf --pdf-outlines --pdf-outlines.headings -o output/slides/12-synthesis-slides.pdf
```

注意: PDF出力にはChromiumが必要。初回実行時に自動ダウンロードされる場合がある。エラーが出た場合は `--browser chrome` オプションを追加してローカルのChromeを使う:

```bash
npx @marp-team/marp-cli output/slides/12-synthesis-slides.md --pdf --pdf-outlines --pdf-outlines.headings --browser chrome -o output/slides/12-synthesis-slides.pdf
```

### 7. 結果の報告

以下を報告する:
- 生成したスライドの枚数
- スライド構成の概要
- 生成されたファイルのパス（.md と .html、PDF選択時は .pdf も）
- 「ブラウザで `output/slides/XXX.html` を開いてください」と案内する
- PDF出力した場合は「`output/slides/XXX.pdf` をダウンロード/共有できます」と案内する
