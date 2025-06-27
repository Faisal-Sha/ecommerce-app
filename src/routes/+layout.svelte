<script>
    let { children } = $props();
    import "../app.css";
    import { cart } from '$lib/cartStore';
    let cartCount = $state(0);
    $effect(() => {
      cart.subscribe(value => {
        cartCount = value.reduce((sum, item) => sum + (item.quantity || 1), 0) || 0;
      });
    });
</script>
{@render children()}
<!-- Removed duplicate cart count badge -->
