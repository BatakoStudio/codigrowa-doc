---
title: デバッグ & 運用
weight: 20
---

エンジンを拡張する際に役立つ検証フローとチェックリストです。

## 1. ログと HUD

- `Game.update` は `__DEV__` 時に `dt` をログ出力します。大量に出る場合は `expo start --no-dev` で実機挙動を確認。
- `Game.onRender` から受け取る `GameRenderPayload` を HUD 表示し、`elapsed` / `tick` / `entities.length` をリアルタイムに監視。`components/game-canvas.tsx` の `functionLogs` 実装を流用できます。

## 2. テスト駆動のすすめ

- `World` と `Entity` は純粋な TypeScript クラスなので Jest で単体テスト可能。
- `Scheduler` は `update(dt)` を段階的に呼び、`task.callback` が発火するかで検証できます。`getTaskCount()` を監視すればリーク検知が容易です。

## 3. 典型的なトラブルシュート

| 症状 | 対処 |
| --- | --- |
| Entity が停止する | `tickRate` が大きすぎて `_process` が呼ばれていない可能性。`__DEV__` ログで `dt` を確認。|
| Renderer に反映されない | `getSnapshot()` で必要なフィールドを返しているか確認。`GameCanvas` 側で型ガードを追加。|
| Scheduler タスクが残り続ける | `game.cancelTask(id)` を `useEffect` のクリーンアップで呼ぶ。|
| 複数画面で Game を共有したい | `useGameStore` の `game` インスタンスをシングルトンとして取り回し、各画面で `onRender` を個別登録。|

## 4. 将来の拡張ポイント

`docs/game_engine_develop_plan.md` に記載されている Step #4 以降（タップ入力、Signal、Sprite 切替など）は現行アーキテクチャを前提としています。API 変更時は互換性を保つため、Hugo ドキュメント内の該当章も更新してください。

## 5. 推奨コマンド

```bash
# Expo プロジェクトで lint
npm run lint

# Hugo ドキュメントのローカルプレビュー
hugo server -D
```

ドキュメントと実装をセットで更新し、他のエンジニアが本リポジトリ（`codigrowa-doc`）を見るだけでエンジンを扱える状態を維持しましょう。
