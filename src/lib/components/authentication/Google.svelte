<script>
	import { GoogleSolid } from 'flowbite-svelte-icons';
	const redirect = () => {
		const clientId = import.meta.env.VITE_GOOGLE_CLIENT_ID;
		const redirectUri = encodeURIComponent(import.meta.env.VITE_GOOGLE_REDIRECT_URI);
		const scope = encodeURIComponent(
			'https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/userinfo.profile'
		);
		const responseType = 'code';
		const accessType = 'offline';
		const prompt = 'select_account';

		const authUrl = `https://accounts.google.com/o/oauth2/v2/auth?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scope}&response_type=${responseType}&access_type=${accessType}&prompt=${prompt}`;

		window.location.href = authUrl;
	};
</script>

<button
	on:click={redirect}
	class="flex gap-2 bg-blue-500 hover:bg-blue-700 text-white font-bold font-mono py-2 px-4 rounded focus:outline-none focus:shadow-outline"
>
	<GoogleSolid class="w-6 h-6" />
	Sign In with Google
</button>
