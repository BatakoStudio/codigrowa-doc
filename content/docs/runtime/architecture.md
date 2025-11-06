---
title: アーキテクチャ概観
weight: 10
---

Codigrowa のゲームエンジンは **GameLoop → Scheduler → World → Entity → Renderer** という鎖で構成されています。Expo（React Native）上では `requestAnimationFrame` を起点に時間が進み、React 側のコンポーネントは `Game.onRender()` から受け取るスナップショットだけで描画を完結させます。

```
useGameLoop (engine/hooks/useGameLoop.ts)
        │
        ▼
┌──────────────┐
│ Game         │  ← ランタイム中枢
│  • Scheduler │  ← 関数単位 tick
│  • World     │  ← Entity 単位 tick
└──────────────┘
        │
        ▼
Canvas Renderer (components/game-canvas.tsx)
```

### データフロー

1. `useGameLoop` が `requestAnimationFrame` から `dt`（秒）を算出し `Game.update(dt)` を連続呼び出し。
2. `Game.update` はまず `Scheduler.update(dt)` を処理し、待機中のコールバックを実行。
3. 続いて `World.process(dt)` が全 `Entity` の `process()` を tick に合わせて回す。
4. `Game.notifyRender()` が World からのスナップショットをまとめ、`onRender` 購読者（HUD や Skia Canvas）へ配布。

描画層は `GameRenderPayload` を受け取るだけで済むため、World/Entity の内部状態を UI から直接 mutate しません。これにより、レンダリングとロジックの責務分離とデバッグ容易性を両立しています。

### 主要ディレクトリ

| ディレクトリ | 役割 |
| --- | --- |
| `engine/Game.ts` | GameLoop とサブシステム統合。start/pause/resume などのライフサイクル API を提供。|
| `engine/World.ts` | Entity 管理・スナップショット生成。|
| `engine/entities/` | Entity 抽象クラスと具象実装（例: `LinearMover`）。|
| `engine/Scheduler.ts` | 時間ベースの関数実行を管理。|
| `engine/hooks/useGameLoop.ts` | React フックで RAF ループを抽象化。|
| `components/game-canvas.tsx` | Game と react-native-skia を橋渡しし、デモ描画や HUD を担う。|

以降の章で各レイヤーの API と実装パターンを詳述します。
