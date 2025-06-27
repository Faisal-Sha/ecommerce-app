# Ecommerce App (Svelte 5)

A modern ecommerce application built with Svelte 5, featuring a reactive UI with cart functionality, product search, pagination, and checkout processes. This app adheres to specific assessment requirements, including the use of Svelte 5 Runes for state management (no Svelte stores) and asynchronous cart processing.

## Features

- **Product Listing and Search**: Fetches products from the Fake Store API (`https://fakestoreapi.com/products`). Includes real-time search with a 300ms debounce and pagination (10 products per page).
- **Cart Functionality**: Asynchronous cart updates using a queue system with a 500ms delay per item. Handles duplicate products by incrementing quantity. Optimistic updates for item removal.
- **Reactive UI**: Cart count badge with an SVG icon and hover effects, total price updates, and queue status indicators ("Adding to cart..." to "Added!"). Uses Svelte 5 Runes (`$state`, `$effect`) for state management.
- **Checkout Process**: Displays cart items with quantities and total price. Includes optimistic deletion, rate limiting (one checkout per 10 seconds with a countdown timer), and a styled success modal.
- **Live Active Users Counter**: Simulates and updates a count of active users every 5 seconds.
- **Data Preloading**: Preloads product data on initial load for faster navigation to the Products page.

## Setup Instructions

1. **Clone the Repository** (if not already done):
   ```bash
   git clone <repository-url>
   cd ecommerce-app
   ```
2. **Install Dependencies**:
   ```bash
   npm install
   ```
3. **Run the Development Server**:
   ```bash
   npm run dev
   ```
   The app will be available at `http://localhost:5173`.

## Testing the Features

1. **Product Listing and Search**:
   - Navigate to the "Products" page.
   - Use the search bar to filter products by title or description. Verify results update in real-time after a short delay.
   - Use pagination controls to view different pages (10 products per page).
   - Note the loading spinner during data fetching and quick navigation due to preloaded data.
2. **Cart Functionality**:
   - Click "Add to Cart" on a product. Observe the button text change to "Adding..." and then back, indicating queue processing.
   - Add the same product multiple times and confirm the quantity increments instead of duplicating entries.
   - Check the cart count badge in the navigation bar updates reactively.
3. **Checkout Process**:
   - Go to the "Checkout" page with items in your cart.
   - Remove an item and confirm the UI updates instantly (optimistic update).
   - Click "Place Order" to checkout. A styled success modal should appear with a confirmation message, auto-closing after 5 seconds or on clicking "Got it".
   - Attempt another checkout immediately. The button should be disabled with a countdown (e.g., "Wait 10s"), and a message should indicate the wait time.
   - Wait 10 seconds and confirm the button re-enables for another checkout.
4. **Live Active Users Counter**:
   - Observe the user count (e.g., "👥 23 users online") in the navigation bar. It should update every 5 seconds with a random number.

## Technical Details

- **Framework**: Svelte 5 with Runes for reactivity (no Svelte stores as per assessment rules).
- **API**: Fake Store API for product data.
- **State Management**: Local state using `$state` and `$effect` for cart, queue, and UI updates.
- **Styling**: Tailwind CSS for responsive and modern UI design.
- **Accessibility**: ARIA labels added for interactive elements like remove buttons.

## Known Issues or Future Improvements

- Enhance error handling for API failures.
- Add more accessibility features like keyboard navigation.
- Implement caching strategies beyond initial preload for even faster navigation.
