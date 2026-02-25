<script lang="ts">
	import { onMount } from 'svelte';
	import KanbanBoard from 'flowbite-svelte/KanbanBoard.svelte';
	import type { KanbanColumnType, KanbanCardType } from 'flowbite-svelte/types';

	const STORAGE_KEY = 'kanban-columns';

	const defaultColumns: KanbanColumnType[] = [
		{
			id: 'todo',
			title: 'Todo',
			cards: [
				{ id: 'card-1', title: 'Task 1', description: 'First task' },
				{ id: 'card-2', title: 'Task 2', description: 'Second task' }
			]
		},
		{
			id: 'in-progress',
			title: 'In Progress',
			cards: [{ id: 'card-3', title: 'Task 3', description: 'Third task' }]
		},
		{
			id: 'done',
			title: 'Done',
			cards: [{ id: 'card-4', title: 'Task 4', description: 'Fourth task' }]
		}
	];

	let columns: KanbanColumnType[] = $state(defaultColumns);

	onMount(() => {
		const saved = localStorage.getItem(STORAGE_KEY);
		if (saved) {
			try {
				columns = JSON.parse(saved);
			} catch {
				// ignore parse errors
			}
		}
	});

	$effect(() => {
		localStorage.setItem(STORAGE_KEY, JSON.stringify(columns));
	});

	function onMove(card: KanbanCardType, from: KanbanColumnType, to: KanbanColumnType) {
		if (from.id === to.id) {
			console.log(`[Same column] Card "${card.title}" reordered in "${from.title}"`);
		} else {
			console.log(`[Cross column] Card "${card.title}" moved from "${from.title}" to "${to.title}"`);
		}
	}
</script>

<div class="min-h-screen bg-gray-100 p-8 dark:bg-gray-900">
	<h1 class="mb-6 text-2xl font-bold text-gray-800 dark:text-white">Kanban Board</h1>
	<KanbanBoard bind:columns {onMove} />
</div>
