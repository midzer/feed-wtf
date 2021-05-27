<script>
    import Header from '$lib/Header/index.svelte';
    import '../app.css';
</script>

<Header />

<main>
    <slot />
</main>

<footer>
    <p><b>ACHTUNG</b>: Frühe ALPHA-Version. Die Sicherstellung der eingetragenen Daten kann nicht garantiert werden!</p>
</footer>
