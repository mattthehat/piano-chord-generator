<script>
    import keys from './data/keys';
    import Header from './UI/Header.svelte';
    import Keyboard from './UI/Keyboard.svelte';
</script>


<Header/>
<main>
	<Keyboard keys={keys} />
</main>

<style>
	main {
        padding: 0 24px 0 24px;
        width:100%;
        max-width: 1280px;
    }
</style>