---
title: GameLoop と状態共有
weight: 50
---

React 側では `engine/hooks/useGameLoop.ts` と `engine/stores/useGameStore.ts` がゲームエンジンを UI に接続します。

## useGameLoop フック

```ts
useGameLoop((dt) => {
  game.update(dt);
}, isRunning && game.isRunning);
```

- `requestAnimationFrame` の `time` から `dt`（秒）を算出。
- `isRunning` が `false` のときは RAF を解除し、`lastTimeRef` をリセット。
- ループ中に最新のコールバック参照を使うため `useRef` で callback を保持。

必要に応じて UI 側から任意の `Game` インスタンスを渡せます。Expo DevTools のホットリロードでも問題なく再初期化されるよう、`useEffect` のクリーンアップで `cancelAnimationFrame` を確実に呼んでいます。

## Zustand ストア

`engine/stores/useGameStore.ts` は `Game` を 1 つだけ生成し、UI からは `start/pause/resume/stop` をディスパッチできるようにしています。

- `setRenderPayload(payload)` が `Game.onRender` から届いたデータを `tick/elapsed/entities` に反映。
- `toggleRunning()` で UI トグルスイッチと同期。
- ストアはクライアントサイドのみで使うため、サーバーレンダリングの心配は不要。

`GameCanvas` 以外の画面（HUD、デバッグパネル、外部ツール）でも同じストアを購読すれば、最新の World スナップショットを共有できます。
