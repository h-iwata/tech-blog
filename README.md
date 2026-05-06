# Tech-Blog Workspace

複数の環境から、Git経由でAgentic RAG（llm-wiki）を活用しながらブログ記事を執筆するためのワークスペースです。

## 全体アーキテクチャ

本プロジェクトは以下のディレクトリ構造で管理されます。

- **`latest/`**: 現在執筆中の記事。タイトルは最後に決めるため、シンプルなファイル名で作業します。
  - `draft.md` (草案)
  - `skeleton.md` (骨格)
  - `article.md` (完成した記事)
- **`archives/`**: 執筆を終えた記事の保存先。
  - `YYYYMMDD-[title]/` のようにフォルダ分けしてアーカイブします。
- **`llm-wiki/`**: 全記事共通のローカルナレッジベース（Agentic RAG用）。
  - `[raw](./llm-wiki/raw)`: スクレイピングしたWebページや公式ドキュメントなどの不変の一次情報。
  - `[wiki](./llm-wiki/wiki)`: AIが抽出・構造化した知識（`concepts`, `entities`, `sources`, `synthesis`）。
- **`docs/`**: AIエージェントおよびユーザー向けのガイドラインとプロンプト。
  - `[blog-guidelines.md](./docs/guidelines/blog-guidelines.md)`: ブログ執筆のルールや品質基準。
  - `[gemini_deep_search_prompt.md](./docs/prompts/gemini_deep_search_prompt.md)`: Gemini Deep Searchなどのプロンプト。
  - `[architecture](./docs/architecture)`: 構成の詳細説明。

## AIエージェントへの指示

複数のAI（Google Antigravity, Claude, OpenCode, Gemini等）が共通のコンテキストを持つため、以下の指示書が用意されています。AIはセッション開始時にこれらを読み込みます。

- [[CLAUDE.md]](./CLAUDE.md)
- [[AGENTS.md]](./AGENTS.md)
- [[GEMINI.md]](./GEMINI.md)
- [[LLM_WIKI_GUIDELINES.md]](./LLM_WIKI_GUIDELINES.md)

## ワークフロー

1. **草案の作成**: `latest/draft.md` に記事のアイディアを書き出します。
2. **骨格の作成**: `latest/skeleton.md` に見出し構成を記述します。
3. **深い調査**: `[gemini_deep_search_prompt.md](./docs/prompts/gemini_deep_search_prompt.md)` を使用し、AIに詳細調査と `llm-wiki/` への知識コンパイルを依頼します。
4. **記事の執筆**: ガイドライン（今後作成）と `llm-wiki/` の知識を用いて `latest/article.md` を執筆します。
5. **公開とアーカイブ**: 完成後、タイトルを決定し `archives/YYYYMMDD-[title]/` に3つのファイルを移動します。
