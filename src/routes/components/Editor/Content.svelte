<script>
	import { autocomplete, notes, note, title } from '$lib/stores';
	import utils from '$lib/utils';
	import { onMount } from 'svelte';
	import { size } from '$lib/stores';

	let editableDiv;
	let oldContent;

	const suggestion = utils.createAutocomplete({ editableDiv });
	const vim = utils.createVim({ editableDiv });

	onMount(() => {
		document.addEventListener('removeSuggestion', suggestion.removeSuggestion);
		document.addEventListener('insertSuggestion', saveNote);
	});

	$: if (editableDiv && $note) {
		oldContent = $note.content;
		editableDiv.innerText = $note.content || '';

		vim.setEditableDiv(editableDiv);
		suggestion.setEditableDiv(editableDiv);
	}

	const saveNote = async () => {
		const deltas = utils.delta(oldContent, editableDiv.innerText);
		await utils.saveNote({ ...$notes[0], deltas });
		oldContent = editableDiv.innerText;
	};

	const handleKeyDown = (event) => {
		vim.keydown(event);
		suggestion.keydown(event);
	};

	const debouncedSaveNote = utils.debounce(async () => {
		await saveNote();
	}, 500);

	const getAutoComplete = async () => {
		if (!$autocomplete || vim.getMode() !== 'insert') return;
		await suggestion.fetchSuggestion($title, editableDiv.innerText);
	};

	const onContentChange = async () => {
		notes.update((allNotes) => {
			const noteIndex = allNotes.findIndex((n) => n.uuid === $note.uuid);
			if (noteIndex > -1) {
				let updatedNote = { ...allNotes[noteIndex], content: editableDiv.innerText };
				allNotes.splice(noteIndex, 1); // Remove the note from its current position
				allNotes.unshift(updatedNote); // Add it to the beginning
			}
			return allNotes;
		});
		debouncedSaveNote();
		await getAutoComplete();
	};
</script>

<div
	role="textbox"
	tabindex="0"
	aria-label="content"
	contenteditable="true"
	bind:this={editableDiv}
	class="editable-div w-full p-4 border-0 border-l border-r border-white h-full bg-gray-900 text-white font-mono {$notes.length !==
	0
		? ''
		: 'pointer-events-none'}"
	placeholder="Start typing..."
	style="font-size: {$size}px;"
	on:input={onContentChange}
	on:keydown={handleKeyDown}
/>

<style>
	.editable-div {
		overflow: auto;
		resize: none;
		outline: none;
		white-space: pre-wrap;
		word-wrap: break-word;
		-webkit-user-modify: read-write-plaintext-only;
	}

	.editable-div[contenteditable='true']:empty:before {
		content: attr(placeholder);
		color: #888;
	}
</style>