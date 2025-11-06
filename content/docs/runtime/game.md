---
title: Game クラス
weight: 20
---

`engine/Game.ts` はランタイムの司令塔です。World と Scheduler を束ね、1 tick ごとに状態更新と描画用スナップショット生成を行います。

## ライフサイクル API

| メソッド | 用途 |
| --- | --- |
| `start({ reset?: boolean })` | GameLoop 起動。`reset` で `elapsed` / `tick` を初期化し即スナップショット配布。|
| `pause()` / `resume()` | `update()` の実行を一時停止／再開。Zustand ストアから UI でトグル。|
| `stop()` | ループ停止 + Scheduler タスク破棄。|
| `update(dt: number)` | `Scheduler.update(dt)` → `World.process(dt)` → `notifyRender()` の順で実行。|

`useGameLoop` から渡される `dt`（秒）だけを頼りに Game は決定的に進行します。React 側で `Game` や `World` を直接触る場合は、状態破壊を避けるためなるべく API 経由（`addEntity`, `wait` など）で操作してください。

## World / Scheduler 依存差し替え

コンストラクタでは `new Game({ world, scheduler })` としてテスト用のモックを注入できます。既存インスタンスを入れ替える場合は `setWorld()` / `setScheduler()` を利用します。

## エンティティ管理ショートカット

- `addEntity(entity: Entity)` / `removeEntity(id)`
- `getWorld()` で低レベル API へアクセス可能（例: デバッグ HUD が `world.getSnapshots()` を直接参照するなど）

World へ登録すると `Entity._onEnterWorld()` が発火し、`_ready()` による初期化フックが 1 度だけ呼ばれます。

## Scheduler の委譲 API

| メソッド | 説明 |
| --- | --- |
| `wait(delay, cb)` | 指定秒数後に 1 度だけ実行。|
| `schedule(cb, interval, { repeat })` | 周期タスク。`repeat: false` で単発化。|
| `cancelTask(id)` | 登録済みタスクを停止。HUD などのアンマウント時に必ず呼ぶ。|
| `clearScheduler()` | 全予約を削除。|

`Game.update` が `Scheduler.update` を必ず先に呼ぶため、Entity 内で `wait/schedule` を使うと **ロジック → 描画** の順序が保証されます。

## レンダリングサブスク

`Game.onRender(cb)` で購読すると以下の `GameRenderPayload` が届きます。

```ts
{
  elapsed: number; // 累計経過秒
  tick: number;    // update 呼び出し回数
  entities: EntitySnapshot[]; // World からの描画向けスナップショット
}
```

`components/game-canvas.tsx` ではこの payload をそのまま Zustand ストアに流しており、Skia Canvas は UI 状態だけで描画を再構築しています。購読解除関数が返るので、React の `useEffect` で登録したら必ずクリーンアップしてください。
