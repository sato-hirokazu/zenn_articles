---
title: "第12章 Agent SkillsでAWS構成図を自動生成する"
---

## はじめに

本章では、以下の記事をもとに、Claude CodeのAgent Skillsを活用したAWS構成図の自動生成実践を整理します。

- 参考記事: https://dev.classmethod.jp/articles/trial-and-error-aws-diagram-agent-skills/
- 記事タイトル: AWS構成図を生成するためにAgent Skillsで試行錯誤してみた

## 1. 背景と目的

AWS構成図を手動で作成するのは時間がかかる作業です。Claude CodeのAgent Skills機能を活用することで、プロンプト一つで構成図を自動生成できるか試行錯誤した取り組みを紹介します。

## 2. Skillsプラグインの導入

### 2.1 Anthropic公式Skillsリポジトリの登録

Anthropicが公開しているSkillsリポジトリをClaude Codeのプラグインとして導入します。

```bash
/plugin marketplace add anthropics/skills
```

### 2.2 example-skillsのインストール

```bash
/plugin install example-skills@anthropic-agent-skills
```

このプラグインに含まれる `skill-creator` を使うことで、Skill作成のガイダンスやテンプレート生成が利用できます。

## 3. Skillのディレクトリ構成

作成したAWS構成図生成Skillのディレクトリ構成は以下の通りです。

```
.claude/skills/aws-architecture-diagram/
├── SKILL.md                              # Skillの定義・ルール
├── references/
│   ├── aws-icons-analytics-ml.md         # AI/ML・分析系アイコン定義
│   ├── aws-icons-common.md               # 共通アイコン定義
│   ├── aws-icons-compute.md              # コンピューティング系アイコン定義
│   ├── aws-icons-networking.md           # ネットワーク系アイコン定義
│   ├── aws-icons-others.md               # その他アイコン定義
│   ├── aws-icons-storage-database.md     # ストレージ・DB系アイコン定義
│   └── layout-guidelines.md              # レイアウトガイドライン
└── scripts/
    └── find_aws_icon.py                  # アイコン検索スクリプト
```

- `SKILL.md` には、AWS構成図を描画する際のルールやレイアウトの指針を記述
- `references/` 配下には、カテゴリ別に分割したAWSアイコンのスタイル定義とレイアウトガイドラインを配置

## 4. 試行錯誤の過程

### 4.1 1回目: まずは生成してみる

簡単なプロンプトで構成図の生成を試みました。

```
Amazon Bedrock Knowledge Bases + Amazon OpenSearch Serverless + Amazon S3 で構成されたRAGシステムの構成図をdraw.ioで作成して。
```

**結果と問題点**:
- Amazon OpenSearch Serverlessが紫の四角形になっていて正しいアイコンが使われていない
- 複数のテキストが被っておりまともに識別できない
- 矢印が縦横に交差していてデータの流れが追いづらい

### 4.2 2回目: アイコン参照の改善

draw.ioで使用できるAWSアイコンの正確なスタイル定義を `references/` 配下にカテゴリ別のMarkdownファイルとしてまとめました。

```markdown
## Storage (#7AA116)

### Service Icons (resourceIcon)

| Icon Name           | resIcon Value                      |
| ------------------- | ---------------------------------- |
| S3                  | `mxgraph.aws4.s3`                  |
| Elastic Block Store | `mxgraph.aws4.elastic_block_store` |
| Glacier             | `mxgraph.aws4.glacier`             |
```

また、必要なアイコンを最小限のトークン消費で検索できるスクリプト `find_aws_icon.py` を作成しました。

**トークン効率の比較**:

| 指標 | referenceの直接読み込み | find_aws_icon.py実行 |
|------|------------------------|---------------------|
| バイト数 | 64,663 bytes | 3,517 bytes |
| 行数 | 1,416 lines | 150 lines |
| 推定トークン数 | ~16,166 tokens | ~879 tokens |

| 指標 | 値 |
|------|-----|
| バイト削減率 | 94.6% |
| 行数削減率 | 89.4% |
| トークン効率 | 18.4倍 |

検索スクリプトを使用することで、アイコン定義ファイルを直接読み込む場合と比較して約95%のトークンを削減できました。

### 4.3 3回目: レイアウトルールの追加

2回目の改善により、アイコンは正しく表示されるようになりました。しかし、まだレイアウトに課題が残っていたため `SKILL.md` にレイアウトルールを追加しました。

```markdown
## レイアウトルール

### グループ化の原則
- AWS Cloudグループを最外層とする
- 機能単位でサブグループを作成（Data Source、Knowledge Base、Vector Store など）
- グループは横並びを基本とし、データフローに沿って配置

### 接続線のルール
- Ingestion Flow: 破線（dashed）で表現
- Query Flow: 実線で表現
- 矢印の向きはデータの流れに合わせる
```

### 4.4 4回目: 色々な構成への対応

最終的には別パターンのアーキテクチャ構成にも対応できるようになりました。微修正は必要なもののドラフト版としては十分です。

以下の要素が適切に表現されています:
- 適切なグループ表現（VPC/Subnet/AZ/AutoScale）
- Application Load Balancer、Amazon ECS、Amazon RDSの3層構造

## 5. 改善のポイント

### 5.1 アイコンの正確な参照が重要

draw.ioのAWSアイコンは独自のスタイル定義が必要です。事前に正確なスタイル情報をSkillに組み込んでおくことで、アイコンの描画ミスを防げます。

また、draw.ioではAmazon OpenSearch ServiceがElasticsearch Serviceという旧名称のままになっており、生成AIが正しいアイコンを見つけられないケースがありました。

### 5.2 レイアウトルールは具体的に

「見やすく配置して」という曖昧な指示では品質が安定しません。グループ化の階層、配置方向、接続線の種類など、具体的なルールを明文化することが重要です。

### 5.3 同じプロンプトで検証を繰り返す

Skillを修正するたびに同じプロンプトで生成結果を比較することで、改善の効果を正確に測定できます。生成AIアプリケーションと同様にInput/Outputのズレを見て修正を繰り返す必要があります。

## 6. 改善サイクル

ワンショットで完璧な構成図が生成できたわけではなく、以下のサイクルを繰り返すことで品質を向上させました。

```
1. Skill作成
   ↓
2. テスト実行
   ↓
3. 問題発見
   ↓
4. SKILL.md修正
   ↓
5. 再テスト
   ↓
(繰り返し)
```

## 7. まとめ

Agent Skillsを活用したAWS構成図の自動生成は、以下の点が重要です。

- **アイコン定義の正確性**: 正しいスタイル情報を事前に準備
- **トークン効率**: 検索スクリプトで必要な情報だけを参照
- **具体的なルール**: レイアウトや接続線のルールを明文化
- **反復的改善**: 同じプロンプトで検証を繰り返す

今後もSkillの改善を続けて、より高品質な構成図を生成できるようにしていくことが重要です。