<script>
    import Header from '$lib/Header/index.svelte';
    import '../app.css';
</script>

<svelte:head>
	<title>feed.wtf</title>
    <link rel="manifest" href="/site.webmanifest">
</svelte:head>

<Header />

<main>
    <slot />
</main>

<footer>
    <p><b>Notice</b>: Early ALPHA version. The security of the entered data can not be guaranteed!</p>
</footer>
