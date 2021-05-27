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
    <p><b>ACHTUNG</b>: Frühe ALPHA-Version. Die Sicherstellung der eingetragenen Daten kann nicht garantiert werden!</p>
</footer>
