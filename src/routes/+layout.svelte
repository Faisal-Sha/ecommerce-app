<script>
    let { children } = $props();
    import "../app.css";
    import { cart } from '$lib/cartStore';
    let cartCount = $state(0);
    $effect(() => {
      cart.subscribe(value => {
        console.log('Cart updated in layout:', value);
        cartCount = value.reduce((sum, item) => sum + (item.quantity || 1), 0) || 0;
      });
    });
</script>
{@render children()}
<span class="bg-red-500 text-white px-2 py-1 rounded-full text-sm">{cartCount}</span>
