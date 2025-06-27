<!-- App.svelte -->
<script>
    import { mount } from 'svelte';
    import { writable } from 'svelte/store';
    import { cart, cartQueue } from '$lib/cartStore';
    
    // Reactive state management using Svelte 5 runes
    let currentPage = $state('home');
    let products = $state([]);
    let isLoading = $state(false);
    let searchTerm = $state('');
    let currentPageNum = $state(1);
    let activeUsers = $state(0);
    let lastCheckoutTime = $state(0);
    let checkoutCooldown = $state(0);
    
    const PRODUCTS_PER_PAGE = 10;
    const DEBOUNCE_DELAY = 300;
    const QUEUE_DELAY = 500;
    const CHECKOUT_COOLDOWN = 10000; // 10 seconds
    
    // Initialize app
    $effect(() => {
      // Simulate active users counter
      const updateActiveUsers = () => {
        activeUsers = Math.floor(Math.random() * 100) + 10;
      };
      updateActiveUsers();
      const interval = setInterval(updateActiveUsers, 5000);
      
      // Update checkout cooldown
      const cooldownInterval = setInterval(() => {
        if (checkoutCooldown > 0) {
          checkoutCooldown = Math.max(0, checkoutCooldown - 1000);
        }
      }, 1000);
      
      return () => {
        clearInterval(interval);
        clearInterval(cooldownInterval);
      };
    });
    
    // Fetch products with preloading
    async function fetchProducts() {
      if (products.length > 0) return; // Already loaded
      
      isLoading = true;
      try {
        const response = await fetch('https://fakestoreapi.com/products');
        products = await response.json();
      } catch (error) {
        console.error('Failed to fetch products:', error);
        // Fallback mock data
        products = Array.from({length: 25}, (_, i) => ({
          id: i + 1,
          title: `Product ${i + 1}`,
          price: (Math.random() * 100 + 10).toFixed(2),
          description: `Description for product ${i + 1}`,
          image: `https://via.placeholder.com/300x300?text=Product+${i + 1}`,
          category: ['electronics', 'clothing', 'books'][i % 3]
        }));
      } finally {
        isLoading = false;
      }
    }
    
    // Debounced search
    let searchTimeout;
    function handleSearch(event) {
      clearTimeout(searchTimeout);
      searchTimeout = setTimeout(() => {
        searchTerm = event.target.value;
        updateURL();
      }, DEBOUNCE_DELAY);
    }
    
    // Cart queue system
    async function processCartQueue() {
      if ($cartQueue.length === 0) return;
      
      const item = $cartQueue[0];
      
      await new Promise(resolve => setTimeout(resolve, 500));
      
      // Update cart and remove from queue
      $cart = [...$cart, { ...item, queueStatus: 'Added to cart!', quantity: 1 }];
      $cartQueue = $cartQueue.slice(1);
      
      if ($cartQueue.length > 0) {
        processCartQueue();
      }
    }
    
    function addToCart(product) {
      const item = { ...product, queueStatus: 'Adding to cart...' };
      $cartQueue = [...$cartQueue, item];
      if ($cartQueue.length === 1) {
        processCartQueue();
      }
    }
    
    function removeFromCart(productId) {
      $cart = $cart.filter(item => item.id !== productId);
    }
    
    function navigate(page, pageNum = 1) {
      currentPage = page;
      if (page === 'home' || page === 'products') {
        searchTerm = ''; // Reset search term on navigation to home or products
      }
      if (page === 'products') {
        currentPageNum = pageNum;
      } else if (page === 'checkout') {
        updateCartPage();
      }
      updateURL();
      
      if (page === 'products') {
        fetchProducts();
      }
    }
    
    function updateURL() {
      const params = new URLSearchParams();
      if (currentPage === 'products') {
        params.set('page', currentPageNum.toString());
        if (searchTerm) params.set('search', searchTerm);
      }
      const newURL = `${window.location.pathname}?${params.toString()}`;
      window.history.replaceState({}, '', newURL);
    }
    
    function checkout() {
      const now = Date.now();
      if (now - lastCheckoutTime < CHECKOUT_COOLDOWN) {
        alert('Please wait before checking out again.');
        return;
      }
      
      lastCheckoutTime = now;
      checkoutCooldown = CHECKOUT_COOLDOWN;
      alert('Checkout successful! Thank you for your purchase.');
      $cart = [];
    }
    
    // Filter and paginate products
    let filteredProducts = $state([]);
    let totalPages = $state(0);
    let paginatedProducts = $state([]);
    
    $effect(() => {
      filteredProducts = products.filter(product => 
        product.title.toLowerCase().includes(searchTerm.toLowerCase()) || 
        product.description.toLowerCase().includes(searchTerm.toLowerCase())
      );
      currentPageNum = 1; // Reset to first page on new search
    });
    
    $effect(() => {
      const pages = Math.ceil(filteredProducts.length / PRODUCTS_PER_PAGE);
      totalPages = pages;
    });
    
    $effect(() => {
      const paginated = filteredProducts.slice(
        (currentPageNum - 1) * PRODUCTS_PER_PAGE,
        currentPageNum * PRODUCTS_PER_PAGE
      );
      paginatedProducts = paginated;
    });
    
    let totalPrice = $state('0.00');
    $effect(() => {
      cart.subscribe(value => {
        totalPrice = value.reduce((sum, item) => {
          return sum + (typeof item.price === "number" ? item.price : 0);
        }, 0).toFixed(2);
      });
    });
    
    let queueStatus = $state('Ready');
    $effect(() => {
      cartQueue.subscribe(value => {
        queueStatus = value.length > 0 ? 'Adding to cart...' : 'Ready';
      });
    });
  </script>
  
  <main class="min-h-screen bg-gray-50">
    <!-- Navigation -->
    <nav class="bg-white shadow-sm border-b">
      <div class="max-w-6xl mx-auto px-4 py-3">
        <div class="flex justify-between items-center">
          <button class="text-2xl font-bold text-gray-800 cursor-pointer" onclick={() => navigate('home')} onkeydown={(e) => { if (e.key === 'Enter' || e.key === ' ') navigate('home'); }}>
            E-Shop
          </button>
          <div class="flex items-center gap-4">
            <span class="text-sm text-gray-600">
              👥 {activeUsers} users online
            </span>
            {#if $cart.length > 0}
              <span class="bg-red-500 text-white px-3 py-1 rounded-full text-sm font-semibold shadow-md transform transition-transform hover:scale-105">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
                </svg>
                {$cart.reduce((sum, item) => sum + (item.quantity || 1), 0)}
              </span>
            {/if}
          </div>
        </div>
      </div>
    </nav>
  
    <!-- Home Page -->
    {#if currentPage === 'home'}
      <div class="max-w-4xl mx-auto px-4 py-16 text-center">
        <h2 class="text-4xl font-bold text-gray-800 mb-8">Welcome to E-Shop</h2>
        <p class="text-xl text-gray-600 mb-12">Discover amazing products at great prices</p>
        
        <div class="flex gap-6 justify-center">
          <button 
            class="bg-blue-600 text-white px-8 py-4 rounded-lg text-lg font-semibold hover:bg-blue-700 transition-colors"
            onclick={() => navigate('products')}
          >
            Products
          </button>
          <button 
            class="bg-green-600 text-white px-8 py-4 rounded-lg text-lg font-semibold hover:bg-green-700 transition-colors"
            onclick={() => navigate('checkout')}
          >
            Checkout ({$cart.length})
          </button>
        </div>
        
        <div class="mt-12 text-sm text-gray-500">
          Cart Status: <span class="font-medium">{queueStatus}</span>
        </div>
      </div>
    {/if}
  
    <!-- Products Page -->
    {#if currentPage === 'products'}
      <div class="max-w-6xl mx-auto px-4 py-8">
        <div class="flex justify-between items-center mb-8">
          <h2 class="text-3xl font-bold text-gray-800">Products</h2>
          <button 
            class="text-blue-600 hover:text-blue-800"
            onclick={() => navigate('home')}
          >
            ← Back to Home
          </button>
        </div>
  
        <!-- Search -->
        <div class="mb-6">
          <input 
            type="text" 
            placeholder="Search products..." 
            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            oninput={handleSearch}
          />
        </div>

        <!-- Loader -->
        {#if isLoading}
          <div class="flex justify-center items-center py-16">
            <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
            <span class="ml-3 text-gray-600">Loading products...</span>
          </div>
        {:else}
          <!-- Products Grid -->
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 mb-8">
            {#each paginatedProducts as product (product.id)}
              <div class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
                <img 
                  src={product.image} 
                  alt={product.title}
                  class="w-full h-48 object-cover"
                  loading="lazy"
                />
                <div class="p-4">
                  <h3 class="font-semibold text-gray-800 mb-2 line-clamp-2">{product.title}</h3>
                  <p class="text-gray-600 text-sm mb-3 line-clamp-2">{product.description}</p>
                  <div class="flex justify-between items-center">
                    <span class="text-xl font-bold text-green-600">${product.price}</span>
                    <button 
                      class="px-3 py-1 bg-blue-600 text-white text-sm rounded hover:bg-blue-700"
                      onclick={() => {
                        addToCart(product);
                      }}
                    >
                      {$cartQueue.some(item => item.id === product.id) ? 'Adding...' : 'Add to Cart'}
                    </button>
                  </div>
                </div>
              </div>
            {/each}
          </div>
  
          <!-- Pagination -->
          {#if totalPages > 1}
            <div class="flex justify-center items-center gap-2">
              <button 
                class="px-3 py-2 rounded border disabled:opacity-50"
                onclick={() => navigate('products', currentPageNum - 1)}
                disabled={currentPageNum === 1}
              >
                Previous
              </button>
              
              {#each Array.from({length: Math.min(5, totalPages)}, (_, i) => {
                const page = Math.max(1, Math.min(totalPages - 4, currentPageNum - 2)) + i;
                return page;
              }) as page}
                <button 
                  class="px-3 py-2 rounded border {page === currentPageNum ? 'bg-blue-600 text-white' : 'hover:bg-gray-100'}"
                  onclick={() => navigate('products', page)}
                >
                  {page}
                </button>
              {/each}
              
              <button 
                class="px-3 py-2 rounded border disabled:opacity-50"
                onclick={() => navigate('products', currentPageNum + 1)}
                disabled={currentPageNum === totalPages}
              >
                Next
              </button>
            </div>
          {/if}
        {/if}
      </div>
    {/if}
  
    <!-- Checkout Page -->
    {#if currentPage === 'checkout'}
      <div class="max-w-4xl mx-auto px-4 py-8">
        <div class="flex justify-between items-center mb-8">
          <h2 class="text-3xl font-bold text-gray-800">Checkout</h2>
          <button 
            class="text-blue-600 hover:text-blue-800"
            onclick={() => navigate('home')}
          >
            ← Back to Home
          </button>
        </div>
  
        {#if $cart.length === 0}
          <div class="text-center py-16">
            <p class="text-xl text-gray-600 mb-4">Your cart is empty</p>
            <button 
              class="bg-blue-600 text-white px-6 py-3 rounded-lg hover:bg-blue-700 transition-colors"
              onclick={() => navigate('products')}
            >
              Start Shopping
            </button>
          </div>
        {:else}
          <!-- Cart Items -->
          <div class="bg-white rounded-lg shadow-md mb-6">
            {#each $cart as item (item.id)}
              <div class="flex items-center gap-4 p-4 border-b last:border-b-0">
                <img 
                  src={item.image} 
                  alt={item.title}
                  class="w-16 h-16 object-cover rounded"
                />
                <div class="flex-1">
                  <h3 class="font-semibold text-gray-800">{item.title}</h3>
                  <p class="text-gray-600">Quantity: {item.quantity}</p>
                </div>
                <div class="text-right">
                  <p class="font-semibold text-gray-800">${(item.price * item.quantity).toFixed(2)}</p>
                  <button 
                    class="text-red-600 hover:text-red-800 text-sm mt-1"
                    onclick={() => removeFromCart(item.id)}
                  >
                    Remove
                  </button>
                </div>
              </div>
            {/each}
          </div>
  
          <!-- Total and Checkout -->
          <div class="bg-white rounded-lg shadow-md p-6">
            <div class="flex justify-between items-center mb-4">
              <span class="text-xl font-semibold">Total:</span>
              <span class="text-2xl font-bold text-green-600">${totalPrice}</span>
            </div>
            
            {#if checkoutCooldown > 0}
              <p class="text-amber-600 mb-4 text-center">
                Please wait {Math.ceil(checkoutCooldown / 1000)} seconds before next checkout
              </p>
            {/if}
            
            <button 
              class="w-full bg-green-600 text-white py-3 rounded-lg text-lg font-semibold hover:bg-green-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
              onclick={checkout}
              disabled={checkoutCooldown > 0}
            >
              {checkoutCooldown > 0 ? `Wait ${Math.ceil(checkoutCooldown / 1000)}s` : 'Complete Checkout'}
            </button>
          </div>
        {/if}
      </div>
    {/if}
  </main>
  
  <style>
    .line-clamp-2 {
      display: -webkit-box;
      -webkit-line-clamp: 2;
      line-clamp: 2;
      -webkit-box-orient: vertical;
      overflow: hidden;
    }
    
    .animate-spin {
      animation: spin 1s linear infinite;
    }
    
    @keyframes spin {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }
  </style>