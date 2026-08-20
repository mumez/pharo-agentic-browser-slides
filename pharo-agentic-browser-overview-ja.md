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
Claude Code、Codex、OpenCode など、複数のAI コーディングエージェントを Pharo から利用できる<strong>統合AIコーディングツール</strong>
</div>

---

<!-- _class: content-image -->

# メインUI

![w:900px](images/agentic-browser.png)

---

# 基本ワークフロー

各AIとのセッションを **トピック** として管理:

1. トピックを作成し、コーディングエージェントを選択
2. リクエストを入力 (`@ClassName` でコード参照、スクリーンショットの添付)
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

AI コーディングエージェント専用の GUI ツールが標準に:

- **Claude Desktop** — ツール、MCP、ファイルアクセスを備えた GUI版のClaude
- **Codex App** — OpenAI の自律的なコーディング環境
- **Cursor / Antigravity / Kiro** — AI ネイティブなエディタ

CUIに比べ、AI エージェントとやり取りする際の敷居が下がった
最初は単純なコード補完やチャットだったが...

---

# 変化の流れ: チャット → マルチエージェントへの委譲

<style scoped>
table { font-size: 26px; }
</style>

| 時代 | パラダイム | 例 |
|-----|----------|---------|
| 黎明期 | エディタサイドバーでのチャット | ChatGPT, GitHub Copilot |
| 少し前 | エージェントモードを持つ AI ファーストな IDE | Cursor, Windsurf |
| 現在 | **タスク全体をエージェントに委譲** | Antigravity 2.0, Codex App |

ツールは日々進化している
もはや「エディタ + チャット」ではなく、UI は **セッションのオーケストレーション** へと向かっている

---

# なぜ Pharo にもこれが必要か

Pharo 開発者も同じパラダイムを享受すべき:

- **リッチなライブ環境** — クラス、メソッド、ランタイムがすぐそこにある
- **[pharo-acp](https://github.com/mumez/pharo-acp)** が複数エージェント向けの ACP クライアントをすでに提供
- **複数プロジェクト**を1つのイメージ内で並行実行可能

<div class="highlight-box">
複数の AI エージェントを制御できる Pharo ネイティブなツールが、次のステップとして自然
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

Pharo 12以上 のイメージで Playground を開き、以下を評価:

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

[ACP](https://agentclientprotocol.com/) 対応であれば、どのエージェントでも利用可能:

| エージェント | インストール |
|-------|---------|
| **Claude Code** | `npm install -g @agentclientprotocol/claude-agent-acp` |
| **Codex** | `npm install -g @agentclientprotocol/codex-acp` |
| **Gemini CLI** | ACP 組み込み (`gemini --acp`) |
| **Copilot CLI** | ACP 組み込み (`copilot --acp --stdio`) |
| **Cursor CLI** | ACP 組み込み (`agent acp`) |

OpenCode、Kilo Code、Kiro CLI などにも対応

> **注意**: Smalltalkコードの質を上げるため 、エージェントには [smalltalk-dev-plugin](https://github.com/mumez/smalltalk-dev-plugin) をインストールしておく

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
最初のメッセージには、Smalltalk開発者スキルを有効化するため、自動的に <code>/st-buddy</code> が付与される (設定で変更可)
</div>

---

<!-- _class: image -->

# トピックの状態

各トピックの状態はステートマシン([SState](https://github.com/mumez/SState))で明確に管理:

![h:520px](images/topic-states.svg)

---

# トピック状態アイコン

| アイコン | 状態 | 意味 |
|------|-------|---------|
| `❇️` | working | AI が作業中 |
| `?` | waitingForHuman | AI の承認待ち |
| `●` | endTurn | ターン終了 |
| `✓` | goalAchieved | ゴール達成 |

---

# Human-in-the-Loop

AI が承認を要求すると:

- ステータスが `?` (waitingForHuman) に変化
- **Send** ボタンが **Allow** に変化
- **Cancel** ボタンが **Deny** に変化

ボタンをクリックして応答すると、AI が再開

<div class="highlight-box">
承認は会話フローの一部として行われる (モーダルダイアログは使わない) 
</div>

---

# コードメンション

チャット内で Pharo のクラスやメソッドを直接参照できる:

```
@QueryClass @DBAdapter>>connect please refactor this
```

AgenticBrowser は各 `@mention` をTonelのソースとして解決し、ACP のテキストリソースにして添付 — **コピペ不要**。

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
サイズの大きいファイルは <code>maxAttachmentSize</code>（デフォルト 5MB）に切り詰められる。送信前にメンションテキストを削除すれば添付はキャンセルされる。
</div>

---

# ゴールの設定

トピックを右クリック → **Set Goal...** で完了条件を入力:

```
all tests pass
```

AgenticBrowser は AI にゴール用のプロンプトを送信。
結果報告ファイル(`result-<topic-id>.md`) が作成されると、トピックは `✓`（`#goalAchieved`）に遷移。

ゴール達成時にはアナウンサーやコールバックブロックのフック(whenGoalAchieved)も発火するので、独自の自動化ワークフローに統合可能。

---

# イメージ内編集監視

トピックに関連するパッケージへの編集をイメージ内で監視:

- トピックを右クリック → **Set Target Packages...** で対象プレフィックスを設定
- 対象のクラス/メソッドがイメージ内で保存されると、ユーザに確認
  - OKなら修正したコードがTonelとしてエクスポートされる
- **追跡対象外**の編集は候補として収集され、後から昇格可能

<div class="highlight-box">
イメージ内でユーザ自身が変更した内容と、AI が見ている Tonel ソースを同期させる仕組み
</div>

---

# セッションの永続化

トピックは Pharo の **Fuel** シリアライザを使って `ab-topics.fuel` に自動保存
イメージ再起動後も利用可能

### 手動での保存・復元:

```smalltalk
AbTopicManager save.
AbTopicManager load.
```
<div class="highlight-box">
トピックごとの状態（設定、ステータス、会話）はすべて永続化される
</div>

---

<!-- _class: section -->
<!-- _paginate: false -->

## カスタマイズ

---

# トピックテンプレート

新規トピック作成時に、作業ディレクトリを指定しない場合、ディレクトリは、`<agenticBrowserRoot>/topic-template` をもとに生成される:

- デフォルトでは `smalltalk-dev-plugin` 向けに調整された `CLAUDE.md` / `AGENTS.md` がコピーされる
- `.claude`、`.opencode` などのエージェント設定用のディレクトリを配置できる
  - スキル、コマンド、ルールを全トピックで共有
- トピックテンプレート全体を独自にカスタマイズしたものに置き換え可能

<div class="highlight-box">
新しいトピックごとにコーディングエージェントを再設定する手間を省ける
</div>

---

# MCP サーバー & エージェントの追加

- **MCP サーバー** — AgenticBrowser のルートに `mcp.json` を配置。組み込みの `smalltalk-interop`/`smalltalk-validator` は自動マージ（`useDefaultMcpServers: false` で無効化可）
- **カスタムエージェント** — Playground または `ab-settings.json` から既存メニューにないコーディングエージェントを登録

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

設定は右クリック → **Edit Settings...** から**トピックごと**にも設定可能

---

<!-- _class: section -->
<!-- _paginate: false -->

## 拡張機能

---

# Web UI（概要）

<div class="highlight-box">
任意の Web ブラウザから <strong>AgenticBrowser</strong> を利用できるオプションパッケージ
</div>

- WebSocket フレームワーク [Ripple](https://github.com/mumez/Ripple) により、リロード不要でリアルタイムに同期
- SolidJS + daisyUI でモバイル端末にも対応
- Spec UI 上とほぼ同じ操作（トピック管理、プロンプト送信、承認など）が可能

**ユースケース**: 
外出先からのスマートフォンでの確認
ディスプレイのないヘッドレス環境からの利用

→ 詳細は [Web UI スライド](https://mumez.github.io/pharo-agentic-browser-slides/pharo-agentic-browser-web-ui-ja.html) を参照

---

<!-- _class: image -->

# Web UI 画面イメージ

![h:520px](images/web-ui-desktop-1.png)

---

# Scripting API（概要）

<div class="highlight-box">
<strong>Smalltalkのスクリプトで AgenticBrowser のトピックをオーケストレーション</strong>できるオプションパッケージ
</div>

- `seq:`、`para:`、`topicBy:`、`agentBy:` などわずか数個のメッセージでマルチエージェントのワークフローを構築
- 結果はステップ間で自動的に受け渡される
  - 手動での情報のやりとりは不要
- AI エージェント自身がスクリプトを書き `st-eval` で実行できる
  - [`ab-scripting-feature-dev`](https://github.com/mumez/pharo-agentic-browser/tree/develop/skills#ab-scripting-feature-dev) スキル

**ユースケース**: 
定型的な AI ワークフローの実行、 CIへの組み込み、複雑なマルチエージェント連携

→ 詳細は [Scripting API スライド](https://mumez.github.io/pharo-agentic-browser-slides/pharo-agentic-browser-scripting-ja.html) を参照

---

# Scripting API — 実際の例

逐次実行ステップ（`seq:`）
— 各トピックの結果は次のトピックのプロンプトに引き継がれる:

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

並行実行ステップ （`para:`）
— トピックは並列実行され、結果は次のステップのためにまとめられる:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder para: {
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk language features.' ].
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk development tools.' ]
    } agentBy: [ :a | a claude ] ].
```

`seq:` と `para:` は自由に組み合わせ可能
 Arenaパターン: [複数のモデルで並行開発 → 結果をまとめて優れたほうを採用](https://mumez.github.io/pharo-agentic-browser-slides/pharo-agentic-browser-scripting-en.html#15)

> - 実践例1: [To-Do アプリ作成スクリプト](https://github.com/mumez/pharo-agentic-browser/blob/develop/docs/to-do-list-orchestration-script.md)
> - 実践例2: [RediStickのTimeSeries対応用スクリプト](https://github.com/mumez/RediStick/blob/master/doc/scripting-features/feature-ts-createrule-deleterule.scripting.md)
---

<!-- _class: section -->
<!-- _paginate: false -->

## まとめ

---

# まとめ

**pharo-agentic-browser** は、コーディングエージェントへの委譲というパラダイムを Pharo にもたらす:

- **ネイティブ GUI** — 複数の AI セッションをPharoから直接管理
- **エージェント非依存** — ACP 対応エージェントであればどれでも利用可能
- **リッチなコンテキスト** — ドラッグ&ドロップでコードメンション、画面キャプチャ
- **Human-in-the-Loop** — 会話の中で承認 (割り込みダイアログなし)
- **ゴール駆動 & 拡張可能** — 完了条件やフックを指定、MCP サーバーやエージェントのカスタマイズが可能
- **Web UI / Scripting API** — ブラウザからの利用や、スクリプトでのオーケストレーションにも対応

---

<!-- _class: all-text-center align-center -->
<!-- _paginate: false -->

# **フィードバックとコントリビューションをお待ちしています！**

https://github.com/mumez/pharo-agentic-browser
