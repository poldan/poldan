<script lang="ts">
    import './layout.css';
    
    // Eliminamos 'export const prerender/ssr' de aquí porque ya están en el .ts
    
    let { children } = $props();
</script>

<svelte:head>
    <link rel="icon" href="/favicon.png" />
</svelte:head>

{@render children()}
