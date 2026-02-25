# Flowbite-Svelte Kanban Board セットアップ計画

## 目的

[flowbite-svelte Kanban Board](https://flowbite-svelte.com/docs/extend/kanban-board) のサンプルを動かす。

- パネル間のカード移動（クロスカラム移動）
- ~~同じパネル内でのカード並び替え（同カラム内移動）~~ → `KanbanBoard` では非対応
- localStorage による状態永続化

## 同じパネル内移動について

**`KanbanBoard` コンポーネントは同カラム内並び替えを対応していない。**

ソースコードを確認したところ、`handleDrop` は同カラムへのドロップを早期リターンするだけで、
カード順序の変更処理は一切実装されていない。`onMove` コールバックも呼ばれない。

- クロスカラム移動: ✅ 対応
- 同カラム内並び替え: ❌ 非対応

同カラム内並び替えを実現したい場合は `KanbanBoard` を使わずカスタム実装が必要。

## 代替手段

### @dnd-kit/svelte（本家公式）

- リポジトリ: [clauderic/dnd-kit](https://github.com/clauderic/dnd-kit)
- バージョン: `0.3.2`（2026年2月19日リリース）
- Svelte 5.29 以上が必要。本家 clauderic が管理する公式 Svelte アダプター。
- `createSortable` プリミティブで同カラム内並び替え・クロスカラム移動の両方に対応。
- React 版と同じ設計思想で、ドキュメントの大部分を共用できる。
- **所見**: v0 のため API 変更リスクはあるが、本家管理・活発な更新・Svelte 5 ネイティブ設計と条件は揃っている。現状最有力。

### SortableJS

- リポジトリ: [SortableJS/Sortable](https://github.com/SortableJS/Sortable)
- Stars: 31k、週間 DL: 128万。npm の最終更新は1年以上前だが、実績と安定性は最高クラス。
- フレームワーク非依存のため、Svelte 5 の `$state` との繋ぎこみを `onEnd` イベントで自前実装する必要がある。
- `KanbanCard` の描画は引き続き使い、カラムの DnD ロジックだけ SortableJS に委ねる構成が現実的。
- **所見**: API は枯れていて安定しているが、Svelte リアクティビティとの統合コードを自前で書く手間がある。実績重視なら選択肢に入る。

## 技術スタック

| 項目 | 選定 |
|---|---|
| Node.js | v24.13.0（mise 管理） |
| フレームワーク | SvelteKit + Vite |
| パッケージマネージャー | pnpm |
| UI ライブラリ | flowbite-svelte |
| CSS | Tailwind CSS v4 + @tailwindcss/vite |
| 永続化 | localStorage |

---

## TODO リスト

### セットアップ

- [x] `.mise.toml` を作成して Node v24.13.0 を固定
  ```toml
  [tools]
  node = "24.13.0"
  ```
- [x] `mise install` で Node を取得・有効化

### プロジェクト作成

- [x] `pnpm dlx sv create .` で SvelteKit scaffold（※ `pnpm create svelte@latest` は deprecated）
  - minimal テンプレートを選択
  - TypeScript を有効化

### 依存パッケージのインストール

- [x] `pnpm add flowbite-svelte`
- [x] `pnpm add -D tailwindcss @tailwindcss/vite`

### 設定ファイルの編集

- [x] `vite.config.ts` に `@tailwindcss/vite` プラグインを追加
- [x] `src/app.css` に `@import "tailwindcss";` を追記（新規作成）
- [x] `src/routes/+layout.svelte` で `app.css` をインポート

### Kanban ページの実装

- [x] `src/routes/+page.svelte` を実装
  - `KanbanBoard`、`KanbanColumnType`、`KanbanCardType` をインポート
  - Todo / In Progress / Done の3カラム初期データを定義
  - `onMove` ハンドラを実装（同パネル内・パネル間を判別してログ出力）
  - `onMount` + `$effect` で localStorage 永続化を実装

### 動作確認

- [ ] `pnpm dev` でサーバーを起動
- [ ] 同カラム内ドラッグでカードの並び順が変わること
- [ ] 別カラムへのドラッグでカードが移動すること
- [ ] リロード後もカード配置が保持されること（localStorage）

---

## 修正対象ファイル一覧

| ファイル | 操作 | 内容 |
|---|---|---|
| `.mise.toml` | 新規作成 | Node v24.13.0 固定 |
| `vite.config.ts` | 編集 | @tailwindcss/vite プラグイン追加 |
| `src/app.css` | 編集 | Tailwind import |
| `src/routes/+layout.svelte` | 編集 | app.css インポート |
| `src/routes/+page.svelte` | 新規作成 | KanbanBoard 実装 |

---

## 教訓

### `pnpm create svelte@latest` は deprecated

`create-svelte` パッケージは非推奨。代わりに `pnpm dlx sv create` を使う。

### flowbite-svelte のインポートパスに注意

`flowbite-svelte/kanban` というサブパスは exports に存在しない。
正しいパス：

- コンポーネント: `flowbite-svelte/KanbanBoard.svelte`
- 型: `flowbite-svelte/types`

### `src/app.css` は scaffold では生成されない

minimal テンプレートでは `app.css` が存在しないため、手動で新規作成が必要。

### flowbite-svelte のスタイルが当たらない（カラムが縦並びになる）

**原因**: flowbite-svelte のテーマは `bg-surface-elevated`、`text-muted` などの独自ユーティリティクラスを使用している。
これらは `flowbite/plugin` を経由して定義されるため、`@import "tailwindcss"` だけでは読み込まれない。
また、Tailwind v4 の content スキャン対象に `node_modules/flowbite-svelte/dist` を含めないと、
コンポーネント内のクラスが purge されてしまう。

**対処**: `flowbite` を dev 依存に追加し、`src/app.css` を以下の内容に更新する。

```css
@import "tailwindcss";
@plugin 'flowbite/plugin';
@custom-variant dark (&:where(.dark, .dark *));
@source "../node_modules/flowbite-svelte/dist";
```

```sh
pnpm add -D flowbite
```

---
