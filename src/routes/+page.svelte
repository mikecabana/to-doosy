<script lang="ts">
	type Todo = {
		id: string;
		label: string;
		done: boolean;
	};

	type Filters = 'all' | 'completed' | 'active';

	let todos = $state<Todo[]>([]);
	let filter = $state<Filters>('all');

	let filteredTodos = $derived(filterTodos(todos, filter));
	let remaining = $derived(getRemainingTodos());

	$effect(() => {
		const savedTodos = localStorage.getItem('todos');
		savedTodos && (todos = JSON.parse(savedTodos));
	});

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));

		console.log(todos);
	});

	function createTodo(event: KeyboardEvent & { currentTarget: EventTarget & HTMLInputElement }) {
		if (event.key !== 'Enter') return;

		const inputEl = event.currentTarget as HTMLInputElement;
		const label = inputEl.value;

		todos = [
			...todos,
			{
				id: crypto.randomUUID(),
				done: false,
				label
			}
		];

		inputEl.value = '';
	}

	function editTodo(event: Event & { currentTarget: EventTarget & HTMLInputElement }) {
		const inputEl = event.currentTarget as HTMLInputElement;
		const id = inputEl.dataset.id!;

		const todo = todos.find((t) => t.id === id);
		if (todo) {
			todo.label = inputEl.value;
		}
	}

	function toggleTodo(event: Event & { currentTarget: EventTarget & HTMLInputElement }) {
		const inputEl = event.currentTarget as HTMLInputElement;
		const id = inputEl.dataset.id!;

		const todo = todos.find((t) => t.id === id);
		if (todo) {
			todo.done = !todo.done;
		}
	}

	function setFilter(newFilter: Filters) {
		filter = newFilter;
	}

	function filterTodos(todos: Todo[], filter: Filters) {
		switch (filter) {
			case 'all':
				return todos;
			case 'active':
				return todos.filter((t) => !t.done);
			case 'completed':
				return todos.filter((t) => t.done);
		}
	}

	function getRemainingTodos() {
		return todos.filter((t) => !t.done).length;
	}
</script>

<svelte:head>
	<link href="https://fonts.googleapis.com/css?family=Press+Start+2P" rel="stylesheet" />
	<link href="https://unpkg.com/nes.css@latest/css/nes.min.css" rel="stylesheet" />
</svelte:head>

<main>
	<div class="container nes-container with-title is-centered">
		<p class="title">TO-DOOSY</p>

		<div class="add-todo nes-field">
			<input onkeydown={createTodo} type="text" class="nes-input" placeholder="Create new todo" />
		</div>

		<div class="todos">
			{#each filteredTodos as todo}
				<div class={'todo-item' + (todo.done ? ' completed' : '')}>
					<div class="nes-field">
						<input
							oninput={editTodo}
							data-id={todo.id}
							value={todo.label}
							type="text"
							class="nes-input"
						/>
					</div>
					<label class="todo-check">
						<input
							onchange={toggleTodo}
							data-id={todo.id}
							checked={todo.done}
							type="checkbox"
							class="nes-checkbox"
						/>
						<span></span>
					</label>
				</div>
			{/each}
		</div>
	</div>

	<div class="filters">
		{#each ['all', 'active', 'completed'] as f}
			<button
				onclick={() => setFilter(f as Filters)}
				class={'nes-btn filter-btn' + (filter === f ? ' is-disabled' : '')}>{f}</button
			>
		{/each}
	</div>

	<div style="margin: 0 1rem">
		{#if remaining > 0}
			{remaining} remaining
		{:else}
			No todos remaining
		{/if}
	</div>
</main>

<style>
	main {
		padding: 1rem;
		height: 100%;
		width: 100%;
		max-width: 500px;
		margin: 0 auto;
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	/*  */

	.container {
		flex-grow: 1;
		min-height: 0;
	}

	.add-todo {
		margin-bottom: 1.25rem;
	}

	.todos {
		height: calc(100% - 2rem - 48px);
		display: flex;
		flex-direction: column;
		gap: 1rem;
		overflow: auto;
		padding-right: 0rem;
		margin-right: -1rem;
		scrollbar-gutter: stable;
	}

	.todo-item {
		position: relative;
		transition: opacity 0.3s;
	}

	.todo-check {
		position: absolute;
		right: 1rem;
		top: 50%;
		transform: translateY(-50%);
	}

	.completed {
		opacity: 0.4;
	}

	/*  */

	.filters {
		display: flex;
		gap: 1rem;
	}

	.filter-btn {
		flex-grow: 1;
		text-transform: capitalize;
	}
</style>
