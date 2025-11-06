---
title: GameCanvas / Renderer
weight: 10
---

`components/game-canvas.tsx` は Game からのスナップショットを `@shopify/react-native-skia` で描画するデモ実装です。エンジンの状態を UI に届ける際のベストプラクティスをまとめます。

## 初期化フロー

1. `startGame({ reset: true })` を `useEffect` で 1 度だけ呼び、GameLoop を開始。
2. `LinearMover` のような Entity をインスタンス化して `game.addEntity()` で登録。
3. クリーンアップ時に `game.removeEntity(entity.id)` を忘れない。

## レンダリング購読

```ts
useEffect(() => {
  const unsubscribe = game.onRender((payload) => {
    setRenderPayload(payload);
  });
  return unsubscribe;
}, [game, setRenderPayload]);
```

Zustand ストアの `entities` を `Canvas` に流し込むことで、UI 層は Game の内部構造に触れずに描画を完了できます。`LinearMoverSnapshot` のように型ガードを挟めば、Canvas 上で扱うデータを限定できます。

## Scheduler を使った HUD

`GameCanvas` では `game.schedule()` と `game.wait()` を使い、定期ログと遅延ログを生成しています。`setFunctionLogs` で最新 4 件だけ表示する仕組みは、Scheduler の動作確認に最適です。

## Skia との結合ポイント

- `Canvas` 内では `entities` のスナップショットをループして `Rect` を描画。
- 描画に必要な情報は `getSnapshot()` の戻り値へ追加する。例: `width`, `height`, `color`。
- UI スレッドから World を直接触らず、描画で使う値はすべてスナップショットから取得する。

## 応用: 別 Renderer への差し替え

同じ `Game.onRender()` を WebGL や Three.js など別のレンダラへ接続しても問題ありません。`Game` はレンダラ非依存なので、**1 つの Game インスタンスを複数 Renderer が購読する** 構成も可能です（HUD + Canvas など）。
