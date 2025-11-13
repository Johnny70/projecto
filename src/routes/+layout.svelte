<script lang="ts">
    import favicon from "$lib/assets/favicon.svg";

    let { children } = $props();
</script>

<svelte:head>
    <link rel="icon" href={favicon} />
</svelte:head>

<main style="height: 100vh; display: flex; flex-direction: column;">
    {@render children()}
</main>
