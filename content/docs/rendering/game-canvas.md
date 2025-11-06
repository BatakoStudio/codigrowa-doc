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

### SharedValue で描画を安定化

Expo SDK 54 時点の `@shopify/react-native-skia` では `useValue` 系フックが廃止されているため、`react-native-reanimated` の `useSharedValue` をそのまま Skia に渡す方式へ移行しました。

- Entity ごとに `useSharedValue` で `x`, `y`, `width`, `height`, `color`, `tapBounds` などを保持するハンドルを作成。
- `game.onRender()` で得たスナップショットをハンドルに流し込み、`.value` を更新しても React の再レンダーは発生しません（Skia が SharedValue を直接参照する）。
- React 側は 4 tick ごとの強制再描画 (`SKIA_PRESENT_INTERVAL`) で HUD テキストだけ更新し、Canvas ノードは常に再利用します。
- `useSharedValue` で管理していれば、Dev Client / 本番ビルドに関係なく滑らかでクラッシュしにくい描画を維持できます。

## 応用: 別 Renderer への差し替え

同じ `Game.onRender()` を WebGL や Three.js など別のレンダラへ接続しても問題ありません。`Game` はレンダラ非依存なので、**1 つの Game インスタンスを複数 Renderer が購読する** 構成も可能です（HUD + Canvas など）。
