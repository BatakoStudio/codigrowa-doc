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
    renderPayloadRef.current = payload;
    setRenderPayload(payload); // HUD 用に tick / elapsed / entityCount だけ更新
  });
  return unsubscribe;
}, [game, setRenderPayload]);
```

`GameCanvas` は `renderPayloadRef` に最新スナップショットを保持しつつ、Zustand には HUD で必要な `tick/elapsed/entityCount` だけを流します。Skia の描画ノードは React ではなく「Entity ID → SharedValue ハンドル」へ直接書き込むため、Game の内部構造を UI から mutate する必要がありません。

## Scheduler を使った HUD

`GameCanvas` では `game.schedule()` と `game.wait()` を使い、定期ログと遅延ログを生成しています。`setFunctionLogs` で最新 4 件だけ表示する仕組みは、Scheduler の動作確認に最適です。

## Skia との結合ポイント

- `renderPayloadRef` を `syncEntityDescriptors()` で走査し、現在生きている Entity ID と種類だけを記録。
- ID ごとに SharedValue をまとめたハンドル（`LinearMoverNode` など）を登録し、`Game.onRender()` から `.value` を更新。
- React は 4 tick ごと (`SKIA_PRESENT_INTERVAL = 4`) に HUD を再描画するだけで、Canvas ノード自体はマウントし直さない。

### SharedValue で描画を安定化

Expo SDK 54 時点の `@shopify/react-native-skia` では `useValue` 系フックが廃止されているため、`react-native-reanimated` の `useSharedValue` をそのまま Skia に渡す方式へ移行しました。

- Entity ごとに `useSharedValue` で `x`, `y`, `width`, `height`, `color`, `tapBounds` などを保持するハンドルを作成。
- `game.onRender()` で得たスナップショットをハンドルに流し込み、`.value` を更新しても React の再レンダーは発生しません（Skia が SharedValue を直接参照する）。
- React 側は 4 tick ごとの強制再描画で HUD テキストだけ更新し、Canvas ノードは常に再利用します。
- `useSharedValue` で管理していれば、Dev Client / 本番ビルドに関係なく滑らかでクラッシュしにくい描画を維持できます。

### 実行環境（Expo Go 非対応）

`@shopify/react-native-skia` は Expo Go には同梱されていません。GameCanvas を動かすには下記のように Dev Client を作成して接続してください。

1. `npm run ios` / `npm run android` で Dev Client（Expo Dev Build）をビルド・インストール。
2. シミュレータ／実機で Dev Client（Codigrowa）を起動し、`npm run start` で Metro Bundler に接続。
3. Expo Go で開いた場合は Skia ネイティブコードが存在しないため、Canvas 初期化時にクラッシュします。

## 応用: 別 Renderer への差し替え

同じ `Game.onRender()` を WebGL や Three.js など別のレンダラへ接続しても問題ありません。`Game` はレンダラ非依存なので、**1 つの Game インスタンスを複数 Renderer が購読する** 構成も可能です（HUD + Canvas など）。
