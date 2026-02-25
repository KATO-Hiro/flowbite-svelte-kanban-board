<script lang="ts">
  import type { KanbanBoardProps, KanbanCardType, KanbanColumnType } from 'flowbite-svelte/types';
  import KanbanCard from 'flowbite-svelte/KanbanCard.svelte';

  let {
    columns = $bindable([]),
    onMove = (_card: KanbanCardType, _from: KanbanColumnType, _to: KanbanColumnType) => {},
    onAddCard = (_col: KanbanColumnType) => {},
    cardProps,
    class: className,
    classes,
    ...restProps
  }: KanbanBoardProps = $props();

  let draggedCard = $state<KanbanCardType | null>(null);
  let sourceColumnId = $state<string | number | null>(null);
  let dragOverColumnId = $state<string | number | null>(null);
  let dragOverCardId = $state<string | number | null>(null);

  function handleDragStart(card: KanbanCardType, colId: string | number) {
    draggedCard = card;
    sourceColumnId = colId;
  }

  function handleDragOver(e: DragEvent, colId: string | number) {
    e.preventDefault();
    dragOverColumnId = colId;
  }

  function handleCardDragOver(e: DragEvent, cardId: string | number) {
    e.stopPropagation();
    e.preventDefault();
    dragOverCardId = cardId;
  }

  function handleDragLeave(e: DragEvent) {
    const currentTarget = e.currentTarget as HTMLElement;
    const relatedTarget = e.relatedTarget as Node | null;
    if (!relatedTarget || !currentTarget.contains(relatedTarget)) {
      dragOverColumnId = null;
    }
  }

  function handleDrop(e: DragEvent, targetColId: string | number) {
    e.preventDefault();
    dragOverColumnId = null;

    if (draggedCard === null || sourceColumnId === null) return;

    const fromCol = columns.find((c) => c.id === sourceColumnId);
    const toCol = columns.find((c) => c.id === targetColId);
    if (!fromCol || !toCol) return;

    if (sourceColumnId === targetColId) {
      // 同カラム内並び替え
      const fromIndex = fromCol.cards.findIndex((c) => c.id === draggedCard!.id);
      const toIndex =
        dragOverCardId !== null
          ? fromCol.cards.findIndex((c) => c.id === dragOverCardId)
          : fromCol.cards.length - 1;

      if (fromIndex !== -1 && toIndex !== -1 && fromIndex !== toIndex) {
        const newCards = [...fromCol.cards];
        const [removed] = newCards.splice(fromIndex, 1);
        newCards.splice(toIndex, 0, removed);
        fromCol.cards = newCards;
        onMove(draggedCard, fromCol, toCol);
      }
    } else {
      // クロスカラム移動
      fromCol.cards = fromCol.cards.filter((c) => c.id !== draggedCard!.id);

      if (dragOverCardId !== null) {
        const toIndex = toCol.cards.findIndex((c) => c.id === dragOverCardId);
        if (toIndex !== -1) {
          toCol.cards = [
            ...toCol.cards.slice(0, toIndex),
            draggedCard!,
            ...toCol.cards.slice(toIndex)
          ];
        } else {
          toCol.cards = [...toCol.cards, draggedCard!];
        }
      } else {
        toCol.cards = [...toCol.cards, draggedCard!];
      }

      onMove(draggedCard, fromCol, toCol);
    }

    draggedCard = null;
    sourceColumnId = null;
    dragOverCardId = null;
  }

  function handleDragEnd() {
    draggedCard = null;
    sourceColumnId = null;
    dragOverColumnId = null;
    dragOverCardId = null;
  }
</script>

<div
  {...restProps}
  class={[
    'grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-3 md:gap-4 p-2 md:p-4',
    className
  ]
    .filter(Boolean)
    .join(' ')}
>
  {#each columns as col (col.id)}
    <div
      role="group"
      aria-label={`${col.title} column drop zone`}
      class={[
        'w-full rounded-xl shadow-sm p-3 md:p-4 flex flex-col bg-surface-elevated text-surface-foreground transition-colors',
        dragOverColumnId === col.id ? 'ring-2 ring-primary' : '',
        classes?.column ?? ''
      ]
        .filter(Boolean)
        .join(' ')}
      ondragover={(e) => handleDragOver(e, col.id)}
      ondragleave={(e) => handleDragLeave(e)}
      ondrop={(e) => handleDrop(e, col.id)}
      style={col.color ? `border-top: 4px solid ${col.color}` : ''}
    >
      <h2
        class={[
          'text-sm md:text-base font-semibold mb-2 md:mb-3 dark:text-white',
          classes?.columnTitle ?? ''
        ]
          .filter(Boolean)
          .join(' ')}
      >{col.title}</h2>

      <div
        class={[
          'flex flex-col gap-2 flex-1 min-h-[60px]',
          classes?.cardList ?? ''
        ]
          .filter(Boolean)
          .join(' ')}
        role="list"
        aria-label={`${col.title} cards`}
      >
        {#each col.cards as card (card.id)}
          <div ondragover={(e) => handleCardDragOver(e, card.id)} role="listitem">
            <KanbanCard
              {card}
              {classes}
              {...cardProps}
              isDragging={draggedCard?.id === card.id}
              onDragStart={() => handleDragStart(card, col.id)}
              onDragEnd={handleDragEnd}
            />
          </div>
        {/each}
      </div>

      <button
        class={[
          'mt-2 md:mt-3 w-full bg-primary text-primary-foreground rounded-lg py-1.5 text-xs md:text-sm dark:text-primary-500 font-medium hover:bg-primary/90 transition-colors focus:ring-2 focus:ring-primary focus:ring-offset-2',
          classes?.addButton ?? ''
        ]
          .filter(Boolean)
          .join(' ')}
        onclick={() => onAddCard(col)}
        aria-label={`Add card to ${col.title}`}
      >+ Add card</button>
    </div>
  {/each}
</div>
