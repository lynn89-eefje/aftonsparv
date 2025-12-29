<script>
	import favicon from '$lib/assets/favicon.svg';

	import "$lib/styles.css";

	let { children } = $props();
</script>
<style>
	#spacing {
		padding-top: 30px;
	}
</style>

<svelte:head>
	<link rel="icon" href={favicon} />

	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Cal+Sans&family=Montserrat:ital,wght@0,100..900;1,100..900&family=Space+Grotesk:wght@300..700&display=swap" rel="stylesheet">
	<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@24,400,0,0&icon_names=add_circle,cancel,check_circle,lock" />
</svelte:head>

<div id="spacing">
	{@render children()}
</div>
