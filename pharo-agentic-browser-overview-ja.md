---
marp: true
theme: classmethod
paginate: true
---

<style>
/* Section divider slides: larger heading */
section.section h2 {
  font-size: 56px;
}
</style>

<!-- _class: title -->
<!-- _paginate: false -->

<style scoped>
section {
  justify-content: center;
  align-items: center;
  gap: 12px;
  text-align: center;
}
section > h1:first-child {
  position: static !important;
  width: auto !important;
  height: auto !important;
  padding-left: 0 !important;
  font-size: 72px;
}
section > h1:first-child::after {
  display: none !important;
}
</style>

# pharo-agentic-browser 概要

### **Pharoの統合AIコーディング環境**
梅澤 真史
https://github.com/mumez/pharo-agentic-browser

---

<!-- _class: section -->
<!-- _paginate: false -->

<style>
.highlight-box {
  margin-top: 32px;
  background-color: #e8f0fe;
  border-left: 6px solid var(--blue-very-deep);
  padding: 24px 32px;
  border-radius: 0 8px 8px 0;
  font-size: 30px;
}
</style>

## pharo-agentic-browser とは？

---

<!-- _class: content-image -->

# pharo-agentic-browser

<div class="highlight-box">
Claude Code、Codex、OpenCode など、複数のAI コーディングエージェントを Pharo から利用できる<strong>統合AIコーディングツール</strong>です。
</div>

---

<!-- _class: content-image -->

# メインUI

![w:900px](images/agentic-browser.png)

---

# 基本ワークフロー

各AIとのセッションは **トピック** として管理します:

1. トピックを作成し、ACP 対応エージェントを選択
2. リクエストを入力 + `@ClassName` でコードを参照、スクリーンショットの添付も可
3. AI が自律的に作業（タスク分解、コード変更、テスト）
4. AI が承認を必要とする場合、チャット内で一時停止して確認
5. プルダウンで応答すると AI が再開
6. トピックの状態は常にサイドバーで確認可能

---

<!-- _class: section -->
<!-- _paginate: false -->

## 開発の動機

---

# コーディングエージェントツールのGUI化

AI コーディングエージェント専用の GUI ツールが標準になりつつあります:

- **Claude Desktop** — ツール、MCP、ファイルアクセスを備えた GUI版のClaude
- **Codex App** — OpenAI の自律的なコーディング環境
- **Cursor / Antigravity / Kiro** — AI ネイティブなエディタ

これらのツールは、AI エージェントとやり取りする際の敷居を下げ、単純なチャットを超えたものになっています。

---

# 変化の流れ: チャット → マルチエージェントへの委譲

<style scoped>
table { font-size: 26px; }
</style>

| 時代 | パラダイム | 例 |
|-----|----------|---------|
| 黎明期 | エディタサイドバーでのチャット | ChatGPT, GitHub Copilot |
| 少し前 | エージェントモードを持つ AI ファーストな IDE | Cursor, Windsurf |
| 現在 | **タスク全体をエージェントに委譲** | Antigravity 2.0, Codex Desktop |

ツールは進化しています。UI はもはや「エディタ + チャット」ではなく、**セッションのオーケストレーション** へと向かっています。

---

# なぜ Pharo にもこれが必要か

Pharo 開発者も同じパラダイムを享受すべき:

- **リッチなライブ環境** — クラス、メソッド、ランタイムがすぐそこにある
- **pharo-acp** が複数エージェント向けの ACP クライアントをすでに提供
- **複数プロジェクト**を1つのイメージ内で並行実行可能

<div class="highlight-box">
複数の AI エージェントを制御できる Pharo ネイティブなツールが、次のステップとして自然です。
</div>

---

# 汎用ツールに対する主な優位性

| 優位性 | 詳細 |
|-----------|-------|
| **コンテキストを直接渡せる** | `@ClassName`、`@Class>>method`、スクリーンショット — コピペ不要 |
| **エージェント非依存** | pharo-acp 経由でさまざまなエージェントと連携 |
| **Pharo との統合** | テスト、コードの変更監視がライブイメージの中で利用可能 |
| **並行セッション** | 複数エージェントを異なるトピックで同時実行 |

---

<!-- _class: section -->
<!-- _paginate: false -->

## インストール

---

# インストール

Pharo 12+ のイメージで Playground を開き、以下を評価:

```smalltalk
Metacello new
    baseline: 'AgenticBrowser';
    repository: 'github://mumez/pharo-agentic-browser:main/src';
    load: 'all'.
```

AgenticBrowserを開く:

```smalltalk
AgenticBrowser open.
```

---

# インストール — エージェント側

ACP 対応であれば、どのエージェントも利用可能:

| エージェント | インストール |
|-------|---------|
| **Claude Code** | `npm install -g @agentclientprotocol/claude-agent-acp` |
| **Codex** | `npm install -g @agentclientprotocol/codex-acp` |
| **Gemini CLI** | ACP 組み込み (`gemini --acp`) |
| **Copilot CLI** | ACP 組み込み (`copilot --acp --stdio`) |
| **Cursor CLI** | ACP 組み込み (`agent acp`) |

OpenCode、Kilo Code、Kiro CLI などにも対応

> **推奨**: Smalltalkコードの質を上げるため 、エージェントに [smalltalk-dev-plugin](https://github.com/mumez/smalltalk-dev-plugin) をインストールしておく

---

<!-- _class: section -->
<!-- _paginate: false -->

## 基本機能

---

# トピックの作成

1. **+ New Topic** をクリック
2. タイトルを入力し、コーディングエージェントを選択
3. （任意）既存のプロジェクトディレクトリを設定
4. **Create** をクリック — トピックが左サイドバーに表示される
5. （任意）右クリック → **Set Target Packages...** で追跡対象パッケージを設定

<div class="highlight-box">
最初のメッセージには、Smalltalk開発者スキルを有効化するため、自動的に <code>/st-buddy</code> が付与されます。
</div>

---

<!-- _class: image -->

# トピックの状態

各トピックの状態はステートマシンで明確に管理:

![h:520px](images/topic-states.svg)

---

# トピック状態アイコン

| アイコン | 状態 | 意味 |
|------|-------|---------|
| `❇️` | working | AI が作業中 |
| `?` | waitingForHuman | AI が承認を待っている |
| `●` | endTurn | ターン完了 |
| `✓` | goalAchieved | ゴール達成 |

---

# Human-in-the-Loop

AI が承認を要求すると:

- ステータスが `?` (waitingForHuman) に変化
- **Send** ボタンが **Allow** に変化
- **Cancel** ボタンが **Deny** に変化

ボタンをクリックして応答すると、AI が再開します。

<div class="highlight-box">
モーダルダイアログはありません。承認は会話フローの一部です。
</div>

---

# コードメンション

チャット内で Pharo のクラスやメソッドを直接参照できます:

```
@QueryClass @DBAdapter>>connect please refactor this
```

AgenticBrowser は各 `@mention` をTonelのソースとして解決し、ACP のテキストリソースにして添付します — **コピペ不要**。

### ドラッグ&ドロップ

- System Browser から **クラス** をドラッグ → `@ClassName` を挿入
- System Browser から **メソッド** をドラッグ → `@ClassName>>methodName` を挿入

---

# スクリーンキャプチャ

**`[ ]`** ボタンをクリックしてスクリーンショットを添付:

1. ボタンをクリック — カーソルが十字カーソルに変化
2. ドラッグして範囲を選択
3. `@sc-20260528-001.png` のようなメンションが入力欄に挿入される
4. 送信 — PNG が画像リソースとしてプロンプトに添付される

---

# ファイル添付

**+** ボタンをクリックしてディスク上の任意のファイルを添付:

1. ボタンをクリック — ファイル選択ダイアログが開く
2. ファイルを選択 — `[filename]` のようなメンションが入力欄に挿入される
3. 送信 — ファイルの内容がテキストリソースとして添付される

<div class="highlight-box">
サイズの大きいファイルは <code>maxAttachmentSize</code>（デフォルト 5MB）に切り詰められます。送信前にメンションテキストを削除すれば添付はキャンセルされます。
</div>

---

# ゴール設定

トピックを右クリック → **Set Goal...** で完了条件を入力:

```
all tests pass
```

AgenticBrowser は AI にゴール用のプロンプトを送信し、`result-<topic-id>.md` が作成されると、トピックは `✓`（`#goalAchieved`）に遷移します。

ゴール達成時にはアナウンサーやコールバックブロックのフック(whenGoalAchieved)も発火するので、独自の自動化ワークフローに統合できます。

---

# イメージ変更監視

トピックに関連するパッケージへの編集をイメージ内で監視:

- トピックを右クリック → **Set Target Packages...** で対象プレフィックスを設定
- 対象のクラス/メソッドが保存されると、確認の上でパッケージがエクスポートされる
- **追跡対象外**の編集は候補として収集され、後から昇格可能

<div class="highlight-box">
イメージ内でユーザ自身が変更した内容と、AI が見ている Tonel ソースとを同期させ続けます。
</div>

---

# セッションの永続化

トピックは Pharo の **Fuel** シリアライザを使って `ab-topics.fuel` に自動保存され、イメージ再起動後も残ります。

手動での保存・復元:

```smalltalk
AbTopicManager save.
AbTopicManager load.
```

トピックごとの状態（設定、ステータス、会話）はすべて永続化されます。

---

<!-- _class: section -->
<!-- _paginate: false -->

## カスタマイズ

---

# トピックテンプレート

新規トピックの作業ディレクトリは、`<agenticBrowserRoot>/topic-template` をもとに生成されます:

- デフォルトでは `smalltalk-dev-plugin` 向けに調整された `CLAUDE.md` / `AGENTS.md` を同梱
- `.claude`、`.opencode` などのエージェント設定ディレクトリを配置できる — スキル、コマンド、ルールを全トピックで共有
- 独自にカスタマイズしたものに置き換え可能

<div class="highlight-box">
新しいトピックごとにコーディングエージェントを再設定する手間を省けます。
</div>

---

# MCP サーバー & カスタムエージェントの追加

- **MCP サーバー** — AgenticBrowser のルートに `mcp.json` を配置。組み込みの `smalltalk-interop`/`smalltalk-validator` は自動マージ（`useDefaultMcpServers: false` で無効化可）
- **カスタムエージェント** — Playground または `ab-settings.json` からメニューにないコーディングエージェントを登録

```smalltalk
AbSettings default codingAgents: (AbSettings default codingAgents copyWith:
    {'name' -> 'my-agent'.
    'command' -> #('my-agent' '--acp')} asDictionary).
AbSettings save.
```

---

# グローバル設定（一部抜粋）

| キー | デフォルト | 説明 |
|-----|---------|-------------|
| `useDefaultMcpServers` | `true` | 組み込みの Smalltalk MCP サーバーをマージ |
| `aiPermissionWaitTimeoutSeconds` | `1800` | 人間の承認待ちタイムアウト |
| `aiPermissionTimeoutOption` | `#reject_once` | 自動応答: `allow_once`, `allow_always`, `reject_once` |

設定は右クリック → **Edit Settings...** から**トピックごと**にも設定可能です。

---

<!-- _class: section -->
<!-- _paginate: false -->

## その他のインターフェース

---

# Web UI（概要）

<div class="highlight-box">
任意の Web ブラウザから <strong>AgenticBrowser</strong> を利用できるオプションパッケージです。
</div>

- WebSocket フレームワーク [Ripple](https://github.com/mumez/Ripple) により、リロード不要でリアルタイムに同期
- SolidJS + daisyUI でモバイル端末にも対応
- Spec UI にある操作（トピック管理、プロンプト送信、承認）はひととおり利用可能

**ユースケース**: 外出先からのスマートフォン確認、ディスプレイのないヘッドレス環境からのアクセスなど

→ 詳細は [Web UI スライド](https://mumez.github.io/pharo-agentic-browser-slides/pharo-agentic-browser-web-ui-ja.html) を参照

---

<!-- _class: image -->

# Web UI 画面イメージ

![h:520px](images/web-ui-desktop-1.png)

---

# Scripting API（概要）

<div class="highlight-box">
UI操作なしに<strong>Smalltalkコードから AgenticBrowser のトピックをオーケストレーション</strong>できるオプションパッケージです。
</div>

- `seq:`、`para:`、`topicBy:`、`agentBy:` などわずか数個のメッセージでマルチエージェントのワークフローを構築
- 結果はステップ間で自動的に受け渡される — 手動での情報のやりとりは不要
- AI エージェント自身がスクリプトを書いて `st-eval` で実行することも可能（`ab-scripting-feature-dev` スキル）

**ユースケース**: 定型的な AI ワークフローの実行や CI 組み込み、ヘッドレス環境での実行、複雑なマルチエージェント連携

→ 詳細は [Scripting API スライド](https://mumez.github.io/pharo-agentic-browser-slides/pharo-agentic-browser-scripting-ja.html) を参照

---

# Scripting API — 実際の例

逐次ステップ（`seq:`）— 各トピックの結果は次のトピックのプロンプトに引き継がれる:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder seq: {
        builder topicBy: [ :t |
            t prompt: 'List 3 Pharo Smalltalk features in one sentence each.' ].
        builder topicBy: [ :t |
            t prompt: 'Summarize the feature list from the previous step in one sentence.' ]
    } agentBy: [ :a | a claude ] ].
```

---

# Scripting API — 並列ステップの例

トピックは並行実行され、結果は次のステップのためにまとめられる:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder para: {
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk language features.' ].
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk development tools.' ]
    } agentBy: [ :a | a claude ] ].
```

`seq:` と `para:` は自由に組み合わせ可能
 — 例: 並列調査して結果をまとめる → 結果をもとに逐次実行

> 実践例: To-Do アプリ
> [to-do-list-orchestration-script.md](https://github.com/mumez/pharo-agentic-browser/blob/develop/docs/to-do-list-orchestration-script.md)

---

<!-- _class: section -->
<!-- _paginate: false -->

## まとめ

---

# まとめ

**pharo-agentic-browser** は、コーディングエージェントへの委譲というパラダイムを Pharo にもたらします:

- **ネイティブ GUI** — イメージから離れずに複数の AI セッションを管理
- **エージェント非依存** — ACP 対応エージェントであればどれでも利用可能
- **リッチなコンテキスト** — コードメンション、ドラッグ&ドロップ、スクリーンキャプチャ
- **Human-in-the-Loop** — 会話の中で承認、割り込みダイアログなし
- **ゴール駆動 & 拡張可能** — 完了条件やフック指定、MCP サーバー/エージェントのカスタマイズに対応
- **Web UI / Scripting API** — ブラウザからの利用や、コード駆動のオーケストレーションにも対応

---

<!-- _class: all-text-center align-center -->
<!-- _paginate: false -->

# **フィードバックとコントリビューションをお待ちしています！**

https://github.com/mumez/pharo-agentic-browser
