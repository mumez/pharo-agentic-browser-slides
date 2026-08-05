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

# pharo-agentic-browser Scripting API

### **エージェントのオーケストレーションをSmalltalkで**
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

## Scripting API とは？

---

<!-- _class: content-image -->

# 概要

<div class="highlight-box">
UI操作なしに<strong>Smalltalkコードから AgenticBrowser のトピックをオーケストレーション</strong>できるオプションパッケージです。
</div>

- 簡単なSmalltalk スクリプトを書くだけで、AgenticBrowser がトピックを作成し、エージェントを実行。結果を受け渡し、すべてが終わるまで待つなどができる
- Spec UI、Web UI に続く3つ目のインターフェース

---

# ユースケース

- **定型的な AI ワークフロー** — 複数ステップからなるワークフローを毎日実行、あるいは CI に組み込む
- **ヘッドレス環境** — ディスプレイがない環境でもトピックを構築・実行
- **複雑なマルチエージェント連携** — UIによる手作業では組みにくい、複雑なエージェントの連携や結果の収集
- **AI によるオーケストレーション記述** — シンプルなDSLにより、コーディングエージェント自身がスクリプトを作成し `st-eval` で実行できる

---

<!-- _class: section -->
<!-- _paginate: false -->

## 機能

---

# シンプルな DSL

- 構築に必要なメッセージは `seq:`、`para:`、`topicBy:`、`agentBy:` のわずか数個
- 人間はもちろん、**AI エージェント自身**でも一回の `eval` で書いて実行できるほど平易
- 結果はステップ間で自動的に受け渡される — 手動でのやりとりは不要

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder seq: {
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk features.' ].
        builder topicBy: [ :t | t prompt: 'Write a one-sentence summary of previous features.' ].
    } agentBy: [ :a | a claude ] ].
```

---

# オーケストレーショングループ

- 1つのオーケストレーション内のトピックだけでなく、**複数のオーケストレーション**を連携させる
- 複数のオーケストレーションを `seq:` / `para:` でネスト可能
- グループの中にグループをネスト可能
- **Arenaパターン**（同じタスクを異なるエージェントに実行させ、最良の結果を選ぶ）や、複数の調査結果を1つの統合ステップにまとめるといったパターンを実現

---

# Fuel による永続化

- オーケストレーション（およびグループ）は Pharo の Fuel シリアライザで**保存・読み込み**可能
- 実行前に保存 — 構築済みの設定を後で再利用
- 実行後に保存 — 後で読み込んでの確認や、中断していた場合は `resume` が可能

---

<!-- _class: section -->
<!-- _paginate: false -->

## インストール

---

# インストール

コアパッケージと合わせて Scripting パッケージをロード:

```smalltalk
Metacello new
    baseline: 'AgenticBrowser';
    repository: 'github://mumez/pharo-agentic-browser:main/src';
    load: 'Scripting'.
```

テストスイートもロードする場合:

```smalltalk
Metacello new
    baseline: 'AgenticBrowser';
    repository: 'github://mumez/pharo-agentic-browser:main/src';
    load: 'Scripting-Tests'.
```

---

<!-- _class: section -->
<!-- _paginate: false -->

## スクリプト例

---

# 単一トピック

`runBy:` はオーケストレーションを構築してすぐに実行し、完了までブロック:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder seq: {
        builder topicBy: [ :t |
            t title: 'List Pharo features'.
            t prompt: 'List 3 Pharo Smalltalk features in one sentence each.' ]
    } agentBy: [ :a | a claude ] ].
```

`scriptBy:` は実行せずに構築だけを行い、後で確認したり `forkRun` したりが可能

---

# 逐次ステップ — `seq:`

各トピックの結果は、次のトピックのプロンプトに引き継がれる:

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

# 並列ステップ — `para:`

トピックは並行実行され、結果は次のステップのためにまとめられる:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder para: {
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk language features.' ].
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk development tools.' ]
    } agentBy: [ :a | a claude ] ].
```

`seq:` と `para:` は自由に組み合わせ可能 — 例: 並列調査 → 逐次統合

---

# オーケストレーショングループ — Arena パターン

異なるエージェントで2つの候補を実行し、勝者を選ぶ:

```smalltalk
AgenticBrowser groupRunBy: [ :groupBuilder |
    groupBuilder para: {
        groupBuilder orchestrationBy: [ :b |
            b seq: { b topicBy: [ :t | t prompt: 'Implement feature XXX.'. t goal: 'all tests pass' ] }
            agentBy: [ :a | a claude ] ].
        groupBuilder orchestrationBy: [ :b |
            b seq: { b topicBy: [ :t | t prompt: 'Implement feature XXX.'. t goal: 'all tests pass' ] }
            agentBy: [ :a | a codex ] ]
    }.
    groupBuilder singleTopicBy: [ :t | t prompt: 'Pick the best implementation above and open a PR.' ]
    agentBy: [ :a | a kilo ]
].
```

---

# 実践例: To-Do アプリ

完全なサンプルはリポジトリのドキュメントを参照:
[to-do-list-orchestration-script.md](https://github.com/mumez/pharo-agentic-browser/blob/develop/docs/to-do-list-orchestration-script.md)

- TDD で完全な **Spec2 To-do リストアプリ**をゼロから構築
- **6エージェント、7フェーズ**: セットアップ → 並列調査 → 設計 → 実装 → UI テスト → レビュー → ドキュメント作成
- `seq:` / `para:`の双方を使用、フェーズごとに軽量モデルとフルモデルを使い分け、実装ステップは`goal:`でテスト全通過まで駆動
- そのままコピペで使えるスクリプト — 1本のオーケストレーションスクリプトがどこまでできるかを示す実例

---

<!-- _class: section -->
<!-- _paginate: false -->

## AI によるオーケストレーション記述

---

# `ab-scripting-feature-dev` スキル

<div class="highlight-box">
<strong>エージェント自身にオーケストレーションスクリプトを書かせる</strong>スキル — 機能を説明するだけで DSL を構築してくれます。
</div>

- 自然言語でプロジェクトに欲しい機能を依頼すると、実行可能なオーケストレーションスクリプトへ変換
- 妥当なフェーズにわけたパイプラインを自動的に構成: **計画（必要な場合）→ TDD 実装 → テスト → リント & スタイルレビュー**
- 生成したスクリプトを `docs/scripting-features/feature-<name>.scripting.md` として、着手させる前にプレビューできる

---

# プレビューしてから実行

- プレビューではスクリプトを**明示的に承認**するまで何も実行されない
- 承認後は、対象リポジトリ（`sharedDirectoryPath:`）に対し `st-eval` 経由で実行
- 実行中は、他のオーケストレーションと同様に **Spec UI** や **Web UI** でリアルタイムに進捗を確認可能

<div class="highlight-box">
ステップが停滞・タイムアウトした場合、スキルは自動的にリトライします。<code>AbOrchestrationManager</code> から手動での確認・リトライも可能です。
</div>

---

# このスキルから生まれた実例: RediStick Time Series

このスキルは単なるデモにとどまらず、実際の機能を生み出した

- [RediStick](https://github.com/mumez/RediStick/) の Time Series サポートは、**このスキルが生成したオーケストレーションスクリプトを実行することで実装**
- 生成されたスクリプト例: [scripting-features/feature-ts-mrange-mrevrange.scripting.md](https://github.com/mumez/RediStick/blob/develop/doc/scripting-features/feature-ts-mrange-mrevrange.scripting.md)
- To-Do アプリの例と同じ構成 — 計画/実装/テスト/レビュー — を実際のプロダクションコードベースに適用

---

<!-- _class: section -->
<!-- _paginate: false -->

## 実行のバリエーション

---

# 同期実行とバックグラウンド実行

**同期実行** — `runBy:` / `script run` はすべて完了するまでブロック

**バックグラウンド実行** — 長時間かかるオーケストレーション向け:

```smalltalk
script forkRunThen: [ :orc | Transcript crShow: 'Done: ' , orc result ].
"... later, if needed:"
script isRunning.
```

`forkRun` は `forkRunThen: [ :orc | ]` の省略形

<div class="highlight-box">
バックグラウンドでオーケストレーションが実行されている間も、<strong>Spec UI</strong> や <strong>Web UI</strong> を開けばリアルタイムに確認できます。thinkingメッセージ、権限確認、トピックの進捗もすべてそこに表示されます。
</div>

---

# キャンセルとレジューム

実行中のオーケストレーションを**キャンセル**:

```smalltalk
script terminate.
```

ステップがタイムアウトした後に**レジューム**（最初の未完了ステップから、記録済みの結果を再利用して再開）:

```smalltalk
script resume.
```

---

<!-- _class: section -->
<!-- _paginate: false -->

## 保存と読み込み

---

# 保存と読み込み

Fuel でオーケストレーション（やグループ）を永続化（実行前/後どちらも可）:

```smalltalk
script save.                                  "-> <sharedDirectoryPath>/<name>.fuel"
script saveTo: '/path/to/my-script.fuel' asFileReference.

loaded := AbTopicOrchestration loadFrom: '/path/to/my-script.fuel'.
loaded result.                                 "results from the saved run"
loaded resume.                                 "continue if the saved run stopped early"
```

<div class="highlight-box">
保存したオーケストレーションは <code>orchestrationLoadFrom:</code> を使って新しいグループにそのまま読み込むこともできます。
</div>

---

<!-- _class: section -->
<!-- _paginate: false -->

## 設定

---

# タイムアウト設定

| 設定 | デフォルト | 適用範囲 |
|---------|---------|-------|
| `orchestrationStepWaitTimeoutSeconds` | 900秒 | トピックのステップごと |
| `orchestrationGroupItemWaitTimeoutSeconds` | 3600秒 | `para:` 内で実行されるグループ項目ごと |

グローバル、またはオーケストレーション/グループごとに調整可能:

```smalltalk
script settings orchestrationStepWaitTimeoutSeconds: 1800.
```

---

# レビュー用にトピックを残す

| 設定 | デフォルト | 効果 |
|---------|---------|--------|
| `lingerOrchestrationTopicsAfterRun` | `false` | オーケストレーション完了後もトピックを Topic Manager に残すかどうか |

- `false` — 実行完了後、トピックは Topic Manager から削除される
- `true` — トピックが残り、後から UI で実行全体の進捗をレビュー可能

```smalltalk
script settings lingerOrchestrationTopicsAfterRun: true.
```

<div class="highlight-box">
Spec UI や Web UI で、結果に至るまでの経緯を後で振り返りたい場合は <code>true</code> に設定します。
</div>

---

<!-- _class: section -->
<!-- _paginate: false -->

## まとめ

---

# まとめ

**Scripting API** は、コード駆動によるヘッドレスなオーケストレーションを AgenticBrowser にもたらす:

- **シンプルな DSL** — `seq:` / `para:` / `topicBy:` / `agentBy:`、人間にも AI エージェントにも扱いやすい
- **オーケストレーショングループ** — ワークフロー全体を並列・逐次・ネストで構成
- **結果のルーティング** — ステップ間の受け渡しは自動。手動の操作は不要
- **柔軟な実行** — 同期/バックグラウンド実行、キャンセル/レジュームに対応
- **Fuel による永続化** — オーケストレーションとその結果を保存・再読み込み

---

<!-- _class: all-text-center align-center -->
<!-- _paginate: false -->

# **フィードバックとコントリビューションをお待ちしています！**

https://github.com/mumez/pharo-agentic-browser
