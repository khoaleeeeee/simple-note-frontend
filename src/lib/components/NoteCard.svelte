<script>
	import stores from '$lib/stores';
	import user from '$lib/stores/user';
	import api from '$lib/api';
	export let note;
	export let activeNote;

	$: highlighted = activeNote && activeNote.uuid === note.uuid;

	const truncateContent = (content, maxLength = 30) => {
		if (!content) return '';
		return content.length > maxLength ? content.substring(0, maxLength) + '...' : content;
	};

	const formatDate = (timestamp) => {
		const date = new Date(timestamp);
		return date.toLocaleDateString('en-US', {
			year: 'numeric',
			month: 'long',
			day: 'numeric',
			hour: '2-digit',
			minute: '2-digit'
		});
	};

	const onDelete = async () => {
		stores.notes.update((notes) => notes.filter((n) => n.uuid !== note.uuid));
		stores.note.set(stores.notes[0]);
		await api.notes.remove({ note, user_uuid: $user.uuid });
	};
</script>

<div
	class="relative p-2 min-h-32 min-w-full border {highlighted
		? 'border-gray-500 bg-gray-500 dark:bg-gray-200 dark:border-gray-500'
		: 'border-gray-300 bg-gray-500 dark:bg-gray-300'} rounded-lg shadow-sm cursor-pointer dark:hover:text-gray-800 dark:hover:bg-gray-400 hover:bg-gray-600 transition-colors duration-300 ease-in-out"
>
	<button
		class="absolute font-bold top-0 right-0 m-2 text-white dark:text-gray-600 hover:text-gray-800"
		on:click={onDelete}
	>
		x
	</button>
	<h2 class="font-bold text-lg text-gray-300 dark:text-gray-800">{note.title}</h2>
	<p class="text-sm text-gray-300 dark:text-gray-600 mt-2">{truncateContent(note.content)}</p>
	<p class="text-xs text-gray-300 dark:text-gray-500 mt-4">{formatDate(note.modified_at)}</p>
</div>
