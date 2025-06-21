# CLAUDE.md

## プロジェクト名

AI駆動開発ウォッチャー（AI-Driven Dev Intelligence Feeder）

## 概要

このリポジトリは、AI駆動型ソフトウェア開発（AI-assisted development）に関する国内外の最新動向を把握するための自動情報収集・要約・投稿システムを提供します。

Reddit、YouTube、Hacker News、Dev.to などのメディア上の投稿から、**特定のキーワード群に基づきクロール・抽出・要約・スコアリング**し、注目度の高いトピックを定期的にSlackへ投稿します。GitHub Actionsを用いた定時ジョブでPythonスクリプトを駆動し、Claudeを活用した要約処理を行います。

---

## Claudeへの期待役割（Role for Claude）

Claudeは、本プロジェクトにおいて以下の知的補助を担います：

* 投稿データから「開発者にとって意味のある」要約文を生成する
* 表層的な言葉だけでなく、背景技術やコンテキストを抽出する
* 類似トピックをグループ化し、重複を避けて代表トピックを出力する
* スコアリングロジックに基づき、上位N件（例：5件）のトピックを要約・整形
* 出力はSlack投稿向けに簡潔・視認性の高い箇条書き形式で整える

---

## 情報抽出・処理フロー

1. **データ収集（Fetch）**

   * Reddit：特定subredditから新規/人気投稿をAPIまたはRSSで取得
   * YouTube：開発関連チャンネルの自動字幕を取得し要約対象に
   * Hacker News：RSSから新着・人気記事を取得
   * Dev.to：RSSを通じてAI関連タグ付き記事を取得

2. **フィルタリング（Filter）**

   * `keywords.txt` をもとに収集対象投稿を選定
   * キーワードは技術名や分類語を含む

3. **クラスタリング（Cluster）**

   * 類似内容を1トピックに集約

4. **ランキング（Score）**

   * スコア = 投稿件数 + エンゲージメント指標（いいね数/コメント数 × 重み）

5. **Claude要約（Summarize）**

   * トピックごとにWhat/Why/Who/Howを要約しMarkdown形式で整形

6. **出力フォーマット（Format）**
   Claudeは以下のMarkdownスタイルで整形してください：

   ```text
   🧠 AI駆動開発まとめ（YYYY年MM月DD日）

   【TOP5 注目トピック】

   Claude Code による自動PRレビュー (Score: 87)
   ClaudeがGitHub Actionsでコードを要約し、自動でPRを生成する仕組みが公開
   出典: https://reddit.com/example1

   FigmaプラグインでUI自動生成 (Score: 79)
   テキストプロンプトからFigmaコンポーネントを生成するAI拡張が話題に
   出典: https://dev.to/example2
   ```

---

## Claudeへのプロンプト制御指針（Prompting Policy）

* **文体**：敬体（〜です・〜ます）で、Slackでの可読性を意識した親しみやすい文調
* **粒度**：各要約は1〜2文以内、背景情報は極力省略
* **フォーマット**：出力フォーマットのMarkdown構文を厳守
* **固有名詞**：ツール名・研究名・モデル名などは省略せず明示
* **締め方**：「〜という投稿がありました」は避け、要点を簡潔に主述で締める
* **例外的掲載**：スコアが低くても注目度・技術的インパクトが高い場合、1件まで例外として含めてよい

---

## スケジューリング仕様（Job Schedule）

* 実行頻度：1日2回（午前8時 / 午後8時）
* 実行方式：GitHub Actions の cron により自動実行
* 出力先：Slackの `#ai-dev-watch` チャンネル（Webhook 経由）

---

## 使用技術・管理体制

* 言語：Python 3.11
* API/RSS連携：

  * Reddit API またはRSS
  * YouTube Data API + 字幕処理
  * Hacker News RSS
  * Dev.to RSS
* LLM：Claude 3 Sonnet または Opus
* Secrets管理：GitHub Secrets または `.env`
* キーワードファイル：`keywords.txt` にて管理・随時更新

---

## セキュリティ・運用方針

* 公開投稿のみを対象とし、個人情報・非公開情報は一切含まない
* Claudeの出力は基本的にそのままSlackに投稿され、事前レビューは行わない
* チームによるリアクションや通報により不適切投稿を検知・削除

---

## 将来的な拡張方針

* Slack以外へのマルチチャネル投稿（Notion, Confluence, Discord など）
* バイリンガル出力（日本語・英語）
* トピックカテゴリ分類（ツール、研究、イベント、採用等）
* 週次・月次の要約生成とアーカイブ化
