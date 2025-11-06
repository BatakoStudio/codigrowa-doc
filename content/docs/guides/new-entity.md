---
title: Entity 実装ガイド
weight: 10
---

このガイドでは新しい Entity を `engine/entities/` に追加し、Game へ組み込むまでの流れを説明します。

## 1. クラス定義

1. `engine/entities/MyEntity.ts` を作成。
2. `Entity` を継承し、`MyEntitySnapshot` 型を定義。
3. コンストラクタで `tickRate` や初期座標を渡す。
4. `_process(elapsed)` に 1 tick 分のロジックを書く。

```ts
import { Entity, EntitySnapshot } from './Entity';

type MyEntitySnapshot = EntitySnapshot & { hp: number };

export class MyEntity extends Entity<MyEntitySnapshot> {
  private hp = 100;

  protected _process() {
    // 例: 徐々に減衰
    this.hp = Math.max(0, this.hp - 1);
  }

  getSnapshot() {
    const { x, y } = this.getPosition();
    return { id: this.id, x, y, hp: this.hp };
  }
}
```

## 2. World へ登録

```ts
const entity = new MyEntity({ tickRate: 1 / 30, x: 50, y: 120 });
game.addEntity(entity);
```

- `Game.addEntity` が World 経由で `_ready()` を一度だけ呼びます。
- 画面遷移やアンマウント時は `game.removeEntity(entity.id)` を必ず実行。

## 3. 描画スナップショットの更新

Skia や React 側が必要とする最小情報だけを `getSnapshot()` に詰めます。大きなオブジェクトを渡すと render 毎にシリアライズコストが掛かるため、描画に不要なロジック用フィールドは含めないでください。

## 4. Scheduler / Signal 連携

- 時間ベースの処理が必要なら、Game をコンストラクタで受け取り `this.game.wait()` を `_ready()` で設定。
- 今後追加予定の Signal システムとも疎結合にできるよう、World への参照が必要な場合は `_ready()` の中で安全にアクセスしましょう。

## 5. デバッグ Tips

- `__DEV__` フラグ下では `Game.update` が `console.log` を出すので挙動確認に有効。
- `Game.onRender` に HUD をぶら下げて `EntitySnapshot` をそのまま表示すると、位置や内部値を即座に確認できます。

これで他の開発者も `Entity` を追加するだけで GameLoop に乗り、Renderer が自動的に新スナップショットを描画してくれるようになります。
