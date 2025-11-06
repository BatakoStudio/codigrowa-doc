---
title: World & Entity
weight: 30
---

`engine/World.ts` と `engine/entities/Entity.ts` はエンジンの心臓部です。World が Entity を束ね、Entity は tickRate ごとに `_process()` を実行します。

## World の責務

- `addEntity(entity)` / `removeEntity(id)` / `clear()` で登録を管理。
- `process(dt)` で全 Entity の `process(dt)` を呼び出し。
- `getSnapshots()` が描画層へ渡す `EntitySnapshot[]` を生成。

World は順序保証のため `Map` を使っています。Entity 追加時に `_onEnterWorld()` が走り、`_ready()` が 1 度だけ実行されます。除去時は `_onExitWorld()` で `_ready()` の再実行を許可します。

## Entity 抽象クラス

```ts
abstract class Entity<TSnapshot extends EntitySnapshot> {
  tickRate: number;      // デフォルト 1/60 秒
  private tickTimer = 0; // 経過時間累積

  process(dt: number) {
    this.tickTimer += dt;
    while (this.tickTimer >= this.tickRate) {
      this.tickTimer -= this.tickRate;
      this._process(this.tickRate);
    }
  }

  protected abstract _process(elapsed: number): void;
  abstract getSnapshot(): TSnapshot;
}
```

### tickRate と決定性

`tickRate` は Entity ごとの更新周期です。World は `dt` をそのまま流すのではなく、`tickTimer` に蓄積した上で tickRate を超えた回数だけ `_process()` を実行します。これにより、RAF のフレームレートが揺らいでもロジックは固定周期で進行します。

### 座標とスナップショット

`Entity` には `setPosition(x, y)` / `getPosition()` などの基本アクセサがあり、`getSnapshot()` で描画層へ公開するデータ構造を自由に定義できます。たとえば `LinearMover` は以下のようなスナップショットを返しています。

```ts
export type LinearMoverSnapshot = EntitySnapshot & {
  x: number;
  y: number;
  width: number;
  height: number;
  color: string;
};
```

## ライフサイクルフック

| フック | タイミング | 用途 |
| --- | --- | --- |
| `_init()` | コンストラクタ内で同期呼び出し | 依存注入や初期値計算。非同期処理は避ける。|
| `_ready()` | World に追加された瞬間に 1 回 | World 参照が必要な初期化（例: Signal 登録）。|
| `_process(elapsed)` | tick ごと | ゲームロジック本体。|
| `_onExitWorld()` | World から削除時 | `_ready` 再実行を許可。リソース解放ここで。|

## 例: LinearMover

`engine/entities/LinearMover.ts` は x 軸をループ移動するデモ Entity です。`tickRate` を個別に指定すると異なる速度・フレームレートでも破綻なく動くことが確認できます。`GameCanvas` では 3 つの LinearMover を登録し、`World.getSnapshots()` 経由で矩形描画しています。
