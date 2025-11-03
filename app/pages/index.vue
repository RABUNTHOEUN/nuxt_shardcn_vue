<template>
  <div class="page">
    <header class="header">
      <div class="brand">ShopSample</div>
      <nav class="nav">
        <a href="#">Home</a>
        <a href="#">Shop</a>
        <a href="#" class="text-gray-600 hover:text-gray-900 px-2 py-1">About</a>
        <a href="#">Contact</a>
      </nav>
      <div class="cart" @click="openCart">
        🛒 <span class="count" v-if="cartCount">{{ cartCount }}</span>
      </div>
    </header>

    <section class="hero">
      <div class="hero-content">
        <h1>Discover your next favorite product</h1>
        <p>Quality items at great prices — curated just for you.</p>
        <div class="hero-ctas">
          <button @click="filterCategory('all')">Shop All</button>
          <button @click="filterCategory('featured')">Featured</button>
        </div>
      </div>
      <img class="hero-image" src="https://picsum.photos/800/400?random=1" alt="hero" />
    </section>

    <section class="controls">
      <div class="search">
        <input v-model="query" placeholder="Search products..." />
      </div>
      <div class="categories">
        <button
          v-for="cat in ['all', ...categories]"
          :key="cat"
          :class="{ active: activeCategory === cat }"
          @click="filterCategory(cat)"
        >
          {{ cat }}
        </button>
      </div>
    </section>

    <section class="products">
      <div class="grid">
        <article class="card" v-for="p in filteredProducts" :key="p.id">
          <img :src="p.image" :alt="p.title" />
          <div class="card-body">
            <h3>{{ p.title }}</h3>
            <p class="price">${{ p.price.toFixed(2) }}</p>
            <p class="desc">{{ p.description }}</p>
            <div class="actions">
              <button @click="addToCart(p)">Add to cart</button>
              <button @click="viewProduct(p)">View</button>
            </div>
          </div>
        </article>
      </div>

      <div v-if="filteredProducts.length === 0" class="empty">
        No products found.
      </div>
    </section>

    <footer class="footer">
      <div>© {{ new Date().getFullYear() }} ShopSample — Built with Nuxt</div>
      <div class="foot-links">
        <a href="#">Privacy</a> · <a href="#">Terms</a>
      </div>
    </footer>

    <div class="cart-modal" v-if="showCart">
      <div class="cart-panel">
        <h2>Your Cart ({{ cartCount }})</h2>
        <ul>
          <li v-for="(item, i) in cart.items" :key="i">
            {{ item.title }} — ${{ item.price.toFixed(2) }}
            <button @click="removeFromCart(i)">×</button>
          </li>
        </ul>
        <div class="cart-total">Total: ${{ cartTotal.toFixed(2) }}</div>
        <div class="cart-actions">
          <button @click="checkout">Checkout</button>
          <button @click="showCart = false">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const products = ref([
  {
    id: 1,
    title: 'Classic Leather Sneakers',
    price: 79.99,
    description: 'Comfortable everyday sneakers.',
    category: 'shoes',
    image: 'https://picsum.photos/300/200?random=11',
    featured: true
  },
  {
    id: 2,
    title: 'Slim Fit Jeans',
    price: 49.99,
    description: 'Stylish denim with stretch.',
    category: 'clothing',
    image: 'https://picsum.photos/300/200?random=12',
    featured: false
  },
  {
    id: 3,
    title: 'Noise-Cancelling Headphones',
    price: 129.99,
    description: 'Immersive sound, all day battery.',
    category: 'electronics',
    image: 'https://picsum.photos/300/200?random=13',
    featured: true
  },
  {
    id: 4,
    title: 'Classic White Tee',
    price: 19.99,
    description: 'A wardrobe essential.',
    category: 'clothing',
    image: 'https://picsum.photos/300/200?random=14',
    featured: false
  },
  {
    id: 5,
    title: 'Running Shorts',
    price: 29.99,
    description: 'Lightweight and breathable.',
    category: 'sports',
    image: 'https://picsum.photos/300/200?random=15',
    featured: false
  },
  {
    id: 6,
    title: 'Smart Watch',
    price: 199.99,
    description: 'Track health and notifications.',
    category: 'electronics',
    image: 'https://picsum.photos/300/200?random=16',
    featured: true
  }
])

const categories = Array.from(new Set(products.value.map(p => p.category)))

const query = ref('')
const activeCategory = ref('all')
const showCart = ref(false)

const cart = ref({
  items: []
})

const cartCount = computed(() => cart.value.items.length)
const cartTotal = computed(() => cart.value.items.reduce((s, i) => s + i.price, 0))

function addToCart(product) {
  cart.value.items.push(product)
  // small visual feedback could be added here
}

function removeFromCart(index) {
  cart.value.items.splice(index, 1)
}

function checkout() {
  alert(`Checkout: ${cartCount.value} items — Total $${cartTotal.value.toFixed(2)}`)
  cart.value.items = []
  showCart.value = false
}

function openCart() {
  showCart.value = true
}

function viewProduct(p) {
  // placeholder — in a real app you'd route to a product page
  alert(`View: ${p.title}`)
}

function filterCategory(cat) {
  activeCategory.value = cat
  if (cat === 'featured') {
    query.value = ''
  }
}

const filteredProducts = computed(() => {
  let list = products.value
  if (activeCategory.value !== 'all' && activeCategory.value !== 'featured') {
    list = list.filter(p => p.category === activeCategory.value)
  }
  if (activeCategory.value === 'featured') {
    list = list.filter(p => p.featured)
  }
  if (query.value.trim()) {
    const q = query.value.toLowerCase()
    list = list.filter(p => p.title.toLowerCase().includes(q) || p.description.toLowerCase().includes(q))
  }
  return list
})
</script>

<style scoped>
.page {
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  color: #222;
  max-width: 1100px;
  margin: 0 auto;
  padding: 16px;
}

.header {
  display: flex;
  align-items: center;
  gap: 16px;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #eee;
}

.brand {
  font-weight: 700;
  font-size: 1.2rem;
}

.nav a {
  margin: 0 8px;
  color: #555;
  text-decoration: none;
}

.cart {
  cursor: pointer;
  position: relative;
}

.count {
  background: #ff4d4f;
  color: #fff;
  padding: 2px 6px;
  border-radius: 999px;
  font-size: 0.8rem;
  margin-left: 6px;
}

.hero {
  display: flex;
  gap: 24px;
  align-items: center;
  margin: 20px 0;
  background: linear-gradient(90deg,#f7fafc,#fff);
  padding: 18px;
  border-radius: 8px;
}

.hero-content {
  flex: 1;
}

.hero h1 {
  margin: 0 0 8px 0;
}

.hero-ctas button {
  margin-right: 8px;
  padding: 8px 12px;
  border: none;
  background: #2563eb;
  color: white;
  border-radius: 6px;
  cursor: pointer;
}

.hero-image {
  width: 320px;
  height: 160px;
  object-fit: cover;
  border-radius: 6px;
}

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 18px 0;
  gap: 12px;
}

.search input {
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  width: 240px;
}

.categories button {
  margin-right: 6px;
  padding: 6px 10px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 6px;
  cursor: pointer;
}

.categories button.active {
  background: #111827;
  color: white;
  border-color: #111827;
}

.products .grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 18px;
}

.card {
  border: 1px solid #eee;
  border-radius: 8px;
  overflow: hidden;
  background: white;
  display: flex;
  flex-direction: column;
}

.card img {
  width: 100%;
  height: 140px;
  object-fit: cover;
}

.card-body {
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  flex: 1;
}

.price {
  font-weight: 700;
  color: #111827;
}

.actions {
  margin-top: auto;
  display: flex;
  gap: 8px;
}

.actions button {
  flex: 1;
  padding: 8px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.actions button:first-child {
  background: #10b981;
  color: white;
}

.actions button:last-child {
  background: #eef2ff;
}

.footer {
  margin: 28px 0 60px;
  padding-top: 18px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  color: #666;
  font-size: 0.9rem;
}

.cart-modal {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.4);
  display: flex;
  align-items: center;
  justify-content: center;
}

.cart-panel {
  background: white;
  padding: 18px;
  width: 360px;
  border-radius: 8px;
}

.cart-panel ul {
  list-style: none;
  padding: 0;
  margin: 8px 0;
  max-height: 220px;
  overflow: auto;
}

.cart-panel li {
  display: flex;
  justify-content: space-between;
  padding: 6px 0;
  border-bottom: 1px solid #f3f3f3;
}

.cart-panel button {
  background: transparent;
  border: none;
  color: #ff4d4f;
  font-weight: 700;
  cursor: pointer;
}

.cart-actions {
  display: flex;
  gap: 8px;
  margin-top: 12px;
}
</style>