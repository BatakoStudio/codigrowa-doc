---
title: Scheduler
weight: 40
---

`engine/Scheduler.ts` は時間ベースの関数実行を担当します。GameLoop の `dt` を受け取り、内部に積んだタスクの `remaining` 時間を減算し、0 を下回ったタイミングで実行します。

## タスクモデル

```ts
type SchedulerTask = {
  id: string;
  callback: () => void;
  remaining: number;
  interval: number;
  repeat: boolean;
};
```

- `wait(delay, callback)` は一度だけ実行するタスクを登録。
- `schedule(callback, interval, { repeat })` はデフォルトで繰り返し（`repeat: true`）。
- `cancel(id)` / `clear()` で後始末。

`Game.update` が Scheduler を World より先に呼び出すため、タスク内で Entity の状態を変更しても同じ tick 内で描画に反映されます。

## 使い所

- HUD へ定期的にログを出す（`components/game-canvas.tsx` 内参照）。
- Cutscene のように「数秒待ってから次アクション」を表現する。
- Entity 内から `this.game.wait()` を呼びたい場合は依存注入で Game インスタンスを受け取り、`_ready()` でタスク登録・ `_onExitWorld()` で `cancelTask()` する。

## エラーハンドリング

タスク実行中に例外が起こっても Scheduler 自身は落ちません。`console.warn()` を出した上で、単発タスクは削除、繰り返しタスクは次のインターバルへ進みます。長時間走り続けるサービスを意識し、repeat タスクでは必ず `cancel()` のタイミングを決めておきましょう。

## パフォーマンスのコツ

- タスク数は `Map` で管理しているため数百件程度までなら問題なし。大量登録が必要なら `getTaskCount()` を監視し、デバッグ HUD から過剰登録を検知する仕組みを追加すると安全です。
- `update(dt)` はタスクが無い場合 O(1) で即 return するため、必要なタイミングでこまめに `cancel()` しておくと無駄なループを避けられます。
