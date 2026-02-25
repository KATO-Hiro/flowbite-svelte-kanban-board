# Flowbite-Svelte Kanban Board セットアップ計画

## 目的

[flowbite-svelte Kanban Board](https://flowbite-svelte.com/docs/extend/kanban-board) のサンプルを動かす。

- パネル間のカード移動（クロスカラム移動）
- 同じパネル内でのカード並び替え（同カラム内移動）
- localStorage による状態永続化

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

## 実装方針

### KanbanBoard をローカルコピーして最小修正（採用）

`flowbite-svelte` の `KanbanBoard.svelte` は同カラム内並び替えに非対応（`handleDrop` が同カラムへのドロップを早期リターンするだけ）。

`src/lib/KanbanBoard.svelte` にコピーし、以下を修正して対応：

- 同カラム早期リターンを削除
- `dragOverCardId` で drop 先カードを追跡
- インデックスベースの `splice` で並び替え
- `onMove` を同カラム時にも呼び出す

外部 DnD ライブラリ不要で変更最小。flowbite 本体の更新は手動で追う必要あり。

---

## TODO リスト（完了）

### フェーズ1: 動作確認

- [x] `pnpm dev` でサーバーを起動して基本動作確認
- [x] 別カラムへのドラッグでカードが移動すること
- [x] リロード後もカード配置が保持されること（localStorage）

### フェーズ2: 同一レーン並び替えの実装

- [x] `flowbite-svelte/dist/kanban/KanbanBoard.svelte` を `src/lib/KanbanBoard.svelte` にコピー
- [x] `handleDrop` の同カラム早期リターンを削除、並び替えロジックを追加
- [x] `src/routes/+page.svelte` のインポートを `$lib/KanbanBoard.svelte` に変更
- [x] 同カラム内ドラッグでカードの並び順が変わることを確認
- [x] クロスカラム移動が引き続き動くことを確認

---

## 教訓

### `pnpm create svelte@latest` は deprecated

代わりに `pnpm dlx sv create` を使う。

### flowbite-svelte のインポートパスに注意

`flowbite-svelte/kanban` は exports に存在しない。正しいパス：

- コンポーネント: `flowbite-svelte/KanbanBoard.svelte`
- 型: `flowbite-svelte/types`

### `src/app.css` は scaffold では生成されない

minimal テンプレートでは手動で新規作成が必要。

### flowbite-svelte のスタイルが当たらない（カラムが縦並びになる）

flowbite-svelte のテーマは `bg-surface-elevated` 等の独自クラスを `flowbite/plugin` 経由で定義している。`@import "tailwindcss"` だけでは読み込まれず、`node_modules/flowbite-svelte/dist` を content スキャン対象に含めないとクラスが purge される。

```css
@import "tailwindcss";
@plugin 'flowbite/plugin';
@custom-variant dark (&:where(.dark, .dark *));
@source "../node_modules/flowbite-svelte/dist";
```

```sh
pnpm add -D flowbite
```

### flowbite-svelte の内部モジュールは直接インポート不可

`clsx`・`tailwind-variants`・`dist/theme/themeUtils` 等はプロジェクト直下にインストールされず TypeScript が解決できない。ローカルコピーしたコンポーネントではこれらへの依存を除去し、クラス文字列をインライン化する。

---
