<script>
    let { children } = $props();
    import "../app.css";
    let cartCount = $state(0);
    // Note: In a real app, we might pass cart data via props or context if needed
    // For now, cartCount will be managed in +page.svelte
</script>
{@render children()}
<!-- Removed duplicate cart count badge -->
