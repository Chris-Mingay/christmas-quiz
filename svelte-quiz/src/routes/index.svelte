<script context="module">
	export const prerender = true;
</script>

<script>
	import { onMount, onDestroy } from 'svelte';
	import Counter from '$lib/Counter.svelte';
	import supabase from '$lib/db';
	import About from './about.svelte';

	let response;
	let rooms;

	async function signUp(){
		const { user, session, error } = await supabase.auth.signUp({
  			email: 'deanlearner@gmail.com',
  			password: 'example-password',
		});
		console.log(user,error);
	}

	async function signIn(){
		const r = await supabase.auth.signIn({
			email: 'deanlearner@gmail.com',
			password: 'example-password',
		});
		response = r;
		console.log(r);
	}

	async function addRoom(){
		const { data, error } = await supabase
			.from('rooms')
			.insert([
				{ name: 'Room Name', size: 'big' }
			]);
	}

	async function getRooms(){
		const { data, error } = await supabase
			.from('rooms')
			.select();
		console.log(data,error);
		rooms = data;
	}

	let mySubscription;

	onMount(async () => {
		console.log("subbing");
		mySubscription = supabase
		.from('rooms')
		.on('INSERT', payload => {
			console.log('Change received!', payload)
		})
		.subscribe();

		console.log(mySubscription);
	});

	onDestroy(() => {
		supabase.removeSubscription(mySubscription);
	});

</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<section>
	<h1>
		<div class="welcome">
			<picture>
				<source srcset="svelte-welcome.webp" type="image/webp" />
				<img src="svelte-welcome.png" alt="Welcome" />
			</picture>
		</div>

		to your new<br />SvelteKit app
	</h1>

	<button on:click={signUp}>Sign Up</button>
	<button on:click={signIn}>Sign In</button>
	<button on:click={addRoom}>Add Room</button>
	<button on:click={getRooms}>Get Rooms</button>

	{#if response}
	<pre>{JSON.stringify(response, null, 2)}</pre>
	{/if}

	{#if rooms}
	<pre>{JSON.stringify(rooms, null, 2)}</pre>
	{/if}

	<h2>
		try editing <strong>src/routes/index.svelte</strong>
	</h2>

	<Counter />
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 1;
	}

	h1 {
		width: 100%;
	}

	.welcome {
		position: relative;
		width: 100%;
		height: 0;
		padding: 0 0 calc(100% * 495 / 2048) 0;
	}

	.welcome img {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		display: block;
	}
</style>
