<script>
	import { onMount, onDestroy, tick } from 'svelte';
	import Edit from '~icons/mdi/edit';
	import Delete from '~icons/mdi/delete-empty';
	import Check from '~icons/mdi/check-bold';
	import Add from '~icons/mdi/add-bold';
	import Sort from '~icons/mdi/sort-calendar-ascending';

	let todos = [];
	let task = '';
	let showFilter = false;
	let showSort = false;
	let sortBy = 'date'; // default sort by date
	let editingId = null;
	let editingTask = '';

	const isBrowser = typeof window !== 'undefined';

	const isLocalStorageAvailable = () => {
		if (!isBrowser) return false;
		try {
			const testKey = '__test__';
			localStorage.setItem(testKey, testKey);
			localStorage.removeItem(testKey);
			return true;
		} catch (e) {
			return false;
		}
	};

	const checkForTodos = () => {
		if (isLocalStorageAvailable()) {
			const storedTodos = localStorage.getItem('todos');
			if (storedTodos) {
				todos = JSON.parse(storedTodos);
			}
		}
	};

	const handleAdd = () => {
		const id = Date.now();
		const finished = false;
		const dateCreated = new Date().toISOString();
		todos = [...todos, { id, finished, task, dateCreated }];
		task = '';
		saveTodosToLocalStorage();
	};

	const handleSortFilter = (btn) => {
		btn === 'sort' ? (showSort = !showSort) : (showFilter = !showFilter);
	};

	const editTodo = (id, newTask) => {
		todos = todos.map((todo) => (todo.id === id ? { ...todo, task: newTask } : todo));
		saveTodosToLocalStorage();
	};

	const updateTodoStatus = (id) => {
		todos = todos.map((todo) => (todo.id === id ? { ...todo, finished: !todo.finished } : todo));
		saveTodosToLocalStorage();
	};

	//extra function to reset local storage, could be used as a delete all button handler
	const handleReset = () => {
		todos = [];
		if (isLocalStorageAvailable()) {
			localStorage.removeItem('todos');
		}
	};

	const handleDelete = (id) => {
		todos = todos.filter((todo) => todo.id !== id);
		saveTodosToLocalStorage();
	};

	const saveTodosToLocalStorage = () => {
		if (isLocalStorageAvailable()) {
			localStorage.setItem('todos', JSON.stringify(todos));
		}
	};

	const sortTodos = () => {
		if (sortBy === 'date') {
			todos = todos.slice().sort((a, b) => new Date(b.dateCreated) - new Date(a.dateCreated));
		} else if (sortBy === 'status') {
			todos = todos.slice().sort((a, b) => a.finished - b.finished);
		}
	};

	const handleSortBy = (sortCriteria) => {
		sortBy = sortCriteria;
		sortTodos();
	};

	const handleEdit = async (id, task) => {
		editingId = id;
		editingTask = task;
		await tick();
		const inputElement = document.getElementById('edit-input');
		if (inputElement) {
			inputElement.focus();
		}
	};

	const handleSaveEdit = (id) => {
		editTodo(id, editingTask);
		editingId = null;
	};

	onMount(() => {
		if (isBrowser) {
			checkForTodos();
			window.addEventListener('beforeunload', saveTodosToLocalStorage);
			sortTodos();
		}
	});

	onDestroy(() => {
		if (isBrowser) {
			saveTodosToLocalStorage();
			window.removeEventListener('beforeunload', saveTodosToLocalStorage);
		}
	});
</script>

<h1 class="title">TO-DO LIST</h1>

<form on:submit|preventDefault={handleAdd} aria-labelledby="add-todo-label">
	<label id="add-todo-label" class="visually-hidden" for="task">Add A To-Do</label>
	<input
		id="task-input"
		bind:value={task}
		name="task"
		autocomplete="off"
		placeholder="Add A To-Do..."
		aria-required="true"
	/>
	<button id="submit-button" type="submit" aria-label="Add Todo"><Add /></button>
</form>

<main>
	<div>
		<div class="section">
			<div class="sort-filter">
				<button
					on:click={() => handleSortFilter('sort')}
					aria-expanded={showSort}
					aria-controls="sort-options"
				>
					Sort <Sort />
				</button>
				{#if showSort}
					<div id="sort-options" class="sort-options">
						<button
							class={sortBy === 'date' ? 'active' : ''}
							on:click={() => handleSortBy('date')}
							aria-pressed={sortBy === 'date'}
						>
							New - to - Old
						</button>
						<button
							class={sortBy === 'status' ? 'active' : ''}
							on:click={() => handleSortBy('status')}
							aria-pressed={sortBy === 'status'}
						>
							By Status
						</button>
					</div>
				{/if}
			</div>

			<ul>
				{#each todos as todo (todo.id)}
					<li>
						<button
							class={todo.finished ? 'finished' : 'complete-btn'}
							on:click={() => updateTodoStatus(todo.id)}
							aria-label={todo.finished ? 'Mark as incomplete' : 'Mark as complete'}
						>
							<Check style="color:white;padding-top:3px" />
						</button>
						{#if editingId === todo.id}
							<input
								id="edit-input"
								bind:value={editingTask}
								on:blur={() => handleSaveEdit(todo.id)}
								aria-labelledby="edit-todo-label"
							/>
						{:else}
							<span id="edit-todo-label" class={todo.finished ? 'finished-text' : ''}
								>{todo.task}</span
							>
						{/if}
						<div class="todo-btns">
							<button on:click={() => handleEdit(todo.id, todo.task)} aria-label="Edit Todo">
								<Edit style="font-size:1.2em; color:darkgray" />
							</button>
							<button on:click={() => handleDelete(todo.id)} aria-label="Delete Todo">
								<Delete style="font-size:1.2em; color:darkgray" />
							</button>
						</div>
					</li>
				{/each}
			</ul>
		</div>
	</div>
</main>

<style>
	.finished {
		background-color: rgb(65, 105, 225);
		border-radius: 100px;
		height: 30px;
		width: 30px;
		border: rgb(65, 105, 225) solid 1.5px;
		margin: 0 0 0 10px;
		cursor: pointer;
	}

	.active {
		background-color: rgb(65, 105, 225) !important;
		color: white;
	}

	.finished-text {
		text-decoration: line-through;
		color: rgb(117, 117, 117);
	}

	.sort-filter > button {
		border: none;
		border-radius: 100px;
		padding: 5px 20px;
		background-color: rgb(234, 241, 251);
		display: flex;
		gap: 5px;
		margin: 0 20px 0 0;
	}

	.sort-filter {
		display: flex;
		border-radius: 100px;
	}

	.sort-options > button {
		background-color: none;
		border: none;
		padding: 5px 20px;
		border-radius: 100px;
		background-color: rgb(234, 241, 251);
	}

	.complete-btn {
		border-radius: 100px;
		height: 30px;
		width: 30px;
		border: darkgray solid 1.5px;
		background-color: white;
		margin: 0 0 0 10px;
		cursor: pointer;
	}

	.complete-btn:hover {
		background-color: darkgray;
	}

	li > span {
		margin: 0 0 0 10px;
		font-family: Arial, Helvetica, sans-serif;
		font-size: 13px;
		width: 60%;
	}

	#edit-input {
		width: 60%;
		border: none;
		margin: 0 0 0 10px;
		outline: none;
	}

	.todo-btns > button {
		border: none;
		background: none;
		cursor: pointer;
		padding: 0;
	}

	.todo-btns {
		width: fit-content;
	}

	form {
		max-width: 800px;
		margin: 0 auto;
		display: flex;
		justify-content: center;
	}

	#task-input {
		border: none;
		border-radius: 100px;
		background-color: rgb(234, 241, 251);
		height: 46px;
		font-size: 16px;
		padding: 0 0 0 30px;
		width: 80%;
		max-width: 500px;
	}

	#task-input:focus-visible {
		background-color: rgb(255, 255, 255);
		box-shadow:
			0 1px 1px 0 rgba(65, 69, 73, 0.3),
			0 1px 3px 1px rgba(65, 69, 73, 0.15);
		-webkit-box-shadow:
			0 1px 1px 0 rgba(65, 69, 73, 0.3),
			0 1px 3px 1px rgba(65, 69, 73, 0.15);
		border: 1px solid transparent;
		outline: none;
	}

	#task-input:focus-visible + #submit-button {
		background-color: rgb(65, 105, 225);
		color: rgb(255, 255, 225);
		cursor: pointer;
	}

	form > button {
		border-radius: 100px;
		border: none;
		margin: 0 0 0 -70px;
		width: 70px;
		font-size: 16px;
	}

	.visually-hidden {
		border: 0;
		clip: rect(0 0 0 0);
		height: 1px;
		margin: -1px;
		overflow: hidden;
		padding: 0;
		position: absolute;
		width: 1px;
	}

	.section {
		width: 80vw;
		max-width: 900px;
		margin: 0 auto;
		height: 400px;
		margin: 30px 0;
	}

	.title {
		text-align: center;
		font-family: Arial, Helvetica, sans-serif;
	}

	main {
		display: flex;
		max-width: 100dvw;
		justify-content: center;
	}

	ul {
		padding: 0;
		margin: 10px 0 0 0;
	}

	li {
		list-style-type: none;
		width: 100%;
		height: 40px;
		display: flex;
		margin: 0 0 10px 0;
		background-color: white;
		border-radius: 10px;
		padding: 0;
		align-items: center;
		justify-content: space-around;
		box-shadow:
			0 1px 1px 0 rgba(65, 69, 73, 0.3),
			0 1px 3px 1px rgba(65, 69, 73, 0.15);
		-webkit-box-shadow:
			0 1px 1px 0 rgba(65, 69, 73, 0.3),
			0 1px 3px 1px rgba(65, 69, 73, 0.15);
		border: 1px solid transparent;
		outline: none;
	}
</style>
