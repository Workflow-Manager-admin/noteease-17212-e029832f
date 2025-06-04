<script>
	// PUBLIC_INTERFACE
	/**
	 * MainContainer for NoteEase: Handles notes CRUD, categorization, and search in a styled light theme.
	 */

	import { onMount } from 'svelte';

	// Basic in-memory state for notes and categories (future: persist, use stores)
	let notes = [];
	let categories = ['All', 'Personal', 'Work', 'Ideas'];
	let selectedCategory = 'All';
	let selectedNoteId = null;
	let searchQuery = '';
	let isEditing = false;
	let newNote = { title: '', content: '', category: 'Personal', id: null };

	// UI state for modal
	let showNoteModal = false;

	// Generate a unique ID for each note
	function generateId() {
		return '_' + Math.random().toString(36).substr(2, 9);
	}

	// PUBLIC_INTERFACE
	function createNote() {
		if (newNote.title.trim() && newNote.content.trim()) {
			const note = {
				...newNote,
				id: generateId(),
				created: new Date(),
				updated: new Date()
			};
			notes = [note, ...notes];
			selectedNoteId = note.id;
			resetModal();
		}
	}

	// PUBLIC_INTERFACE
	function updateNote() {
		if (newNote.title.trim() && newNote.content.trim() && newNote.id) {
			notes = notes.map((n) =>
				n.id === newNote.id ? { ...n, ...newNote, updated: new Date() } : n
			);
			selectedNoteId = newNote.id;
			resetModal();
		}
	}

	// PUBLIC_INTERFACE
	function deleteNote(id) {
		if (confirm('Delete this note?')) {
			notes = notes.filter((n) => n.id !== id);
			if (selectedNoteId === id) selectedNoteId = null;
		}
	}

	// PUBLIC_INTERFACE
	function startEditing(note) {
		newNote = { ...note };
		isEditing = true;
		showNoteModal = true;
	}

	// PUBLIC_INTERFACE
	function startCreating() {
		newNote = { title: '', content: '', category: categories[1] || '', id: null };
		isEditing = false;
		showNoteModal = true;
	}

	// Select a note to show in main area
	function selectNote(id) {
		selectedNoteId = id;
	}

	function resetModal() {
		showNoteModal = false;
		isEditing = false;
		newNote = { title: '', content: '', category: categories[1] || '', id: null };
	}

	// PUBLIC_INTERFACE
	function filterNotes() {
		return notes.filter((n) => {
			const matchCat = selectedCategory === 'All' || n.category === selectedCategory;
			const matchSearch =
				n.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
				n.content.toLowerCase().includes(searchQuery.toLowerCase());
			return matchCat && matchSearch;
		});
	}

	let showSidebar = true;

	// Demo: load with sample notes if first render
	onMount(() => {
		if (notes.length === 0) {
			notes = [
				{
					id: generateId(),
					title: 'Welcome to NoteEase!',
					content: "Click the + button to create a new note.\nTry editing, deleting, or categorizing your notes.",
					category: 'Personal',
					created: new Date(),
					updated: new Date()
				},
				{
					id: generateId(),
					title: 'Work Ideas',
					content: 'Draft the project plan and share with team.',
					category: 'Work',
					created: new Date(),
					updated: new Date()
				}
			];
		}
	});
</script>

<style>
:global(body) {
	background: #F5F7FA;
	color: #222;
	font-family: system-ui, sans-serif;
}
.main-container {
	display: flex;
	flex-direction: column;
	min-height: 100vh;
	background: var(--secondary-bg);
}
.top-bar {
	display: flex;
	justify-content: space-between;
	align-items: center;
	background: #fff;
	padding: 0.75rem 1.5rem;
	box-shadow: 0 1px 6px 0 rgba(60, 60, 80, 0.12);
	border-bottom: 2px solid #F0F3FA;
	position: sticky;
	top: 0;
	z-index: 10;
}
.search-input {
	border: 1.5px solid #4A90E2;
	border-radius: 1.5rem;
	padding: 0.5rem 1.5rem;
	width: 260px;
	font-size: 1rem;
	outline: none;
	background: #F5F7FA;
	color: #2f3542;
}
.brand {
	font-weight: 700;
	color: #4A90E2;
	font-size: 1.3rem;
	letter-spacing: 1.5px;
	display: flex;
	align-items: center;
}
.layout {
	display: flex;
	width: 100%;
	min-height: 0;
	flex: 1 1 0;
}
.sidebar {
	width: 280px;
	background: #fff;
	box-shadow: 1px 0 5px 0 rgba(90,90,140,0.08);
	padding: 1.5rem 0.5rem 1.5rem 1.5rem;
	display: flex;
	flex-direction: column;
	gap: 2.5rem;
	overflow-y: auto;
}
@media (max-width: 700px) {
	.sidebar { width: 64vw; min-width: 180px; }
	.layout { flex-direction: column }
	.main-area { min-height: 40vh; }
}

.categories {
	margin-bottom: 1.6rem;
}
.categories-title {
	font-weight: 600;
	font-size: 1.03rem;
	color: #777;
	margin-bottom: 0.6rem;
	letter-spacing: 0.5px;
}
.category-list {
	list-style: none;
	padding: 0;
	margin: 0;
}
.category-list li {
	margin-bottom: 0.5em;
	cursor: pointer;
	padding: 0.25em 0.8em 0.25em 0.3em;
	font-size: 0.99rem;
	border-radius: 1.5em;
	display: flex;
	align-items: center;
	transition: background 0.14s;
}
.category-list li.selected,
.category-list li:hover {
	background: #eaf4ff;
	color: #185fc9;
	font-weight: 600;
}

.notes-list-title {
	font-size: 1.11rem;
	font-weight: 600;
	color: #4A90E2;
	margin: 0 0 0.8rem 0;
	letter-spacing: 0.4px;
}
.notes-list {
	list-style: none;
	padding: 0;
	margin: 0;
}
.notes-list li {
	padding: 0.67em 0.95em 0.67em 0.5em;
	border-radius: 1.1em;
	margin-bottom: 0.3em;
	cursor: pointer;
	color: #2e3141;
	transition: background 0.13s, color 0.09s;
	font-size: 0.98rem;
	display: flex;
	align-items: center;
	justify-content: space-between;
	background: none;
}
.notes-list li.selected,
.notes-list li:hover {
	background: #F5F7FA;
	color: #185fc9;
	font-weight: 600;
}
.notes-list .note-title {
	max-width: 145px;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
.notes-list .note-cat {
	margin-left: 0.33em;
	font-size: 0.94em;
	color: #abc6ef;
}
.no-notes {
	font-size: 0.96em;
	color: #b8b9bc;
	padding: 0.8em;
}
.main-area {
	flex: 1 1 0;
	padding: 2.7rem 2rem 3.5rem 2rem;
	background: #F5F7FA;
	display: flex;
	flex-direction: column;
	overflow-y: auto;
	border-left: 2px solid #F0F3FA;
	position: relative;
}
.note-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 1.8rem;
}
.note-title {
	font-size: 2.0rem;
	font-weight: 650;
	color: #24375B;
	word-break: break-word;
}
.note-dates {
	color: #abc6ef;
	font-size: 0.93em;
	margin-top: 0.25em;
}
.note-actions button {
	background: none;
	border: none;
	cursor: pointer;
	color: #888;
	font-size: 1rem;
	margin-left: 0.6em;
	padding: 0.3em 0.6em;
	border-radius: 2em;
	transition: background .13s, color .16s;
}
.note-actions button:hover {
	background: #e6eeff;
	color: #185fc9;
}
.note-content {
	font-size: 1.10rem;
	line-height: 1.6;
	color: #404452;
	padding-top: 0.7rem;
	white-space: pre-line;
}
.fab {
	position: fixed;
	bottom: 2rem; right: 2.5rem;
	width: 62px;
	height: 62px;
	border-radius: 50%;
	background: #4A90E2;
	color: #fff;
	font-size: 2rem;
	border: none;
	box-shadow: 0 4px 15px 0 rgba(74,144,226,0.18);
	cursor: pointer;
	display: flex;
	justify-content: center;
	align-items: center;
	transition: background 0.13s;
	z-index: 20;
}
.fab:hover {
	background: #185fc9;
	color: #FFD700;
}
.modal-backdrop {
	position: fixed;
	top: 0; left: 0; right: 0; bottom: 0;
	background: rgba(80, 90, 130, 0.10);
	z-index: 100;
	display: flex;
	align-items: center;
	justify-content: center;
}
.modal {
	background: #fff;
	border-radius: 1.6em;
	padding: 2em 2.2em 1.2em 2.2em;
	box-shadow: 0 7px 28px 5px rgba(74,144,226,0.14);
	width: 95vw;
	max-width: 410px;
}
.modal-title {
	font-size: 1.4rem;
	margin-bottom: 1.3rem;
	color: #4A90E2;
	font-weight: 700;
}
.modal-form label {
	display: block;
	font-size: 1em;
	margin: 0.7em 0 0.25em 0.1em;
	color: #444F5E;
}
.modal-form input,
.modal-form textarea,
.modal-form select {
	width: 100%;
	font-size: 1.02em;
	padding: 0.36em 0.75em;
	border-radius: 0.6em;
	border: 1.1px solid #D7E6FD;
	resize: vertical;
	box-sizing: border-box;
	background: #F5F7FA;
	color: #1e2e4a;
}
.modal-form textarea {
	min-height: 110px;
	max-height: 320px;
}
.form-actions {
	display: flex;
	justify-content: flex-end;
	gap: 1em;
	margin-top: 1.2em;
}
.form-actions button {
	background: #4A90E2;
	color: #fff;
	border: none;
	border-radius: 1.5em;
	font-size: 1em;
	padding: 0.47em 1.3em;
	cursor: pointer;
	font-weight: 600;
	transition: background 0.15s, color 0.12s;
}
.form-actions button.secondary {
	background: #F5F7FA;
	color: #24375B;
	border: 1.1px solid #abc6ef;
}
.form-actions button.secondary:hover {
	background: #eaf4ff;
	color: #185fc9;
}
.form-actions button:hover:not(.secondary) {
	background: #185fc9;
	color: #FFD700;
}
::-webkit-scrollbar {
	width: 8px;
	background: #f5f7fa;
}
::-webkit-scrollbar-thumb {
	background: #eaf1f9;
	border-radius: 6px;
}
::-webkit-scrollbar-thumb:hover {
	background: #abc6ef;
}
</style>

<div class="main-container">
	<div class="top-bar">
		<div class="brand">📝 NoteEase</div>
		<input
			class="search-input"
			type="search"
			placeholder="Search notes..."
			bind:value={searchQuery}
		>
	</div>
	<div class="layout">
		<aside class="sidebar">
			<!-- Categories -->
			<div class="categories">
				<div class="categories-title">Categories</div>
				<ul class="category-list">
					{#each categories as cat}
						<li
							class:selected={selectedCategory === cat}
							on:click={() => selectedCategory = cat}
						>{cat}</li>
					{/each}
				</ul>
			</div>
			<!-- Notes list -->
			<div>
				<div class="notes-list-title">Notes</div>
				<ul class="notes-list">
					{#if filterNotes().length === 0}
						<div class="no-notes">No notes{selectedCategory !== 'All' ? ` in "${selectedCategory}"` : ''}</div>
					{:else}
						{#each filterNotes() as note}
							<li
								class:selected={selectedNoteId === note.id}
								on:click={() => selectNote(note.id)}
							>
								<span class="note-title">{note.title}</span>
								{#if note.category}<span class="note-cat">{note.category}</span>{/if}
							</li>
						{/each}
					{/if}
				</ul>
			</div>
		</aside>
		<main class="main-area">
			{#if selectedNoteId}
				{#if notes.find(n => n.id === selectedNoteId) as note}
					<div class="note-header">
						<div>
							<div class="note-title">{note.title}</div>
							<div class="note-dates">
								Created: {note.created.toLocaleString()} |
								Updated: {note.updated.toLocaleString()}
							</div>
						</div>
						<div class="note-actions">
							<button aria-label="Edit" title="Edit" on:click={() => startEditing(note)}>✏️</button>
							<button aria-label="Delete" title="Delete" on:click={() => deleteNote(note.id)}>🗑️</button>
						</div>
					</div>
					<div class="note-content">{note.content}</div>
				{:else}
					<div style="color:#aab8cc; font-size:1.1rem; margin-top:2.5em;">Note not found.</div>
				{/if}
			{:else}
				<div style="color:#aab8cc; font-size:1.1rem; margin-top:2.5em;">Select or create a note to get started.</div>
			{/if}
		</main>
	</div>
	<!-- Floating Action Button -->
	<button
		class="fab"
		title="Add new note"
		aria-label="Add note"
		on:click={startCreating}
	>
		+
	</button>
	{#if showNoteModal}
		<div class="modal-backdrop" on:click={resetModal}>
			<div class="modal" on:click|stopPropagation>
				<div class="modal-title">{isEditing ? 'Edit Note' : 'New Note'}</div>
				<form class="modal-form" on:submit|preventDefault={isEditing ? updateNote : createNote}>
					<label>Title</label>
					<input type="text" bind:value={newNote.title} required maxlength="100"/>
					<label>Content</label>
					<textarea bind:value={newNote.content} required maxlength="3000"/>
					<label>Category</label>
					<select bind:value={newNote.category}>
						{#each categories.slice(1) as cat}
							<option value={cat}>{cat}</option>
						{/each}
					</select>
					<div class="form-actions">
						<button
							type="button"
							class="secondary"
							on:click={resetModal}
							tabindex="0"
						>Cancel</button>
						<button type="submit">{isEditing ? 'Save' : 'Create'}</button>
					</div>
				</form>
			</div>
		</div>
	{/if}
</div>
