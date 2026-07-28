# UNSPOKEN

スマホ1台を回して遊ぶ、合コン・飲み会向けのパーティゲーム Web アプリ。UI は全て日本語、モバイル縦画面前提（コンテンツ幅は最大 440px）。

## 構成

[index.html](index.html) の1ファイル完結。ビルド工程・パッケージマネージャ・バックエンドは無い。ブラウザで直接開けば動く。

- React 18 / ReactDOM / babel-standalone を CDN から読み込み（バージョン固定）
- JSX は `<script type="text/babel">` 内にそのまま記述
- スタイルは基本インライン style。`@keyframes` と `input` の共通スタイルのみ `<head>` の `<style>` に置く
- 状態は React state のみ。保存・通信・永続化は一切しない（リロードで全消去）

**この単一ファイル構成は意図的なもの**。分割やビルド導入は指示があるまで行わない。

## 画面の流れ

`App` がモード選択を持ち、2つのフローに分岐する。

**MATCH（`SeatFlow`）** — 2択質問の回答一致度で男女をペアにする
`count`（人数）→ `pickQ`（プリセット質問選択・カスタム追加）→ `questions`（内容確認）→ `assign`（番号割り当て）→ `callup` → `answering`（1人ずつ回答）→ `aggregate`（全体集計）→ `pairs`（ペア発表）

**SECRET（`SecretFlow`）** — 質問者側の番号に対して回答者が匿名投票する
`rules` → `role`（どちらが質問するか）→ `count` → `assign` → `question`（質問設定）→ `shownum` → `answering` → `handoff`（次の人へ端末を渡す）→ `result`

各フローは巨大な1コンポーネント内で `if (step === "...") return (...)` を並べる形。ステップを追加するときはこの並びに合わせる。

## ロジックの要点

- `buildPairs()`: 男女の全組み合わせで一致数を採点し、スコア最大のペアを貪欲に確定していく。同点候補は `Math.random()` で選ぶため**結果は毎回変わりうる**
- 質問ごとの `matchMode` は `"same"`（同じ回答でペア）か `"opposite"`（違う回答でペア）
- `PRESET_QUESTIONS` / `SECRET_TEMPLATES` が質問のプリセット。文言はそのまま画面に出るので短く保つ
- `COLORS_M` / `COLORS_F` が男女の識別色。人数がリストより多い場合は剰余で循環する

## SECRET モードの前提

「誰が何番か」を異性に知られると匿名性が崩れる設計。`handoff` / `shownum` の警告表示と画面の出し分けは**ゲームの成立条件そのもの**なので、省略・簡略化しない。
