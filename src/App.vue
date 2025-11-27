<template>
  <div id="app">

    <nav class="floating-nav" :class="{ 'checkout-mode': !showProduct }">
      
      <template v-if="showProduct">
        
        <div class="nav-left">
          <div class="nav-brand">
            <img src="./assets/logo.png" alt="Logo" class="nav-logo" />
          </div>
        </div>

        <div class="nav-right">
          
          <div class="nav-search">
            <span class="fas fa-search search-icon"></span>
            <input
              class="search-input"
              type="text"
              v-model="searchTerm"
              placeholder="Search..."
            />
          </div>

          <div class="nav-filter">
            <button 
              class="filter-btn" 
              :class="{ 'active': showFilters }"
              @click="showFilters = !showFilters"
              title="Filter & Sort"
            >
              <span class="fas fa-sliders-h"></span>
            </button>

            <Transition name="menu-slide">
              <div 
                class="filter-menu" 
                v-if="showFilters" 
                v-click-outside="() => showFilters = false"
              >
                <div class="filter-header">
                  <span>View Settings</span>
                  <button class="close-link" @click="showFilters = false">Done</button>
                </div>
                
                <div class="filter-group">
                  <label>Sort By</label>
                  <CustomDropdown v-model="sortAttribute" :options="sortOptions" />
                </div>

                <div class="filter-group">
                  <label>Order</label>
                  <CustomDropdown v-model="sortOrder" :options="orderOptions" />
                </div>
              </div>
            </Transition>
          </div>

          <div class="nav-actions action-wrapper">
            <button
              @click="toggleCheckout"
              :disabled="cart.length === 0"
              class="checkout-btn-nav icon-mode"
              title="Go to Checkout"
            >
              <span class="fas fa-shopping-cart" style="font-size: 1rem;"></span>
            </button>
            
            <Transition name="badge-pop">
              <span v-if="cartItemCount" class="badge-corner">{{ cartItemCount }}</span>
            </Transition>
          </div>

        </div>
      </template>

      <template v-else>
        <button 
          class="close-checkout-btn" 
          @click="toggleCheckout" 
          title="Return to Lessons"
        >
          <span class="fas fa-times"></span>
        </button>
      </template>

    </nav>

    <main>
      
      <Transition name="fade" mode="out-in" @after-enter="onAfterEnter">

        <div v-if="showProduct">

          <section class="hero-section">
            <div class="hero-visual">
              <img src="./assets/hero.png" alt="Classroom Illustration" />
            </div>
            <div class="hero-content">
              <h1>Welcome to <br><span class="brand-highlight">After Bell Corner</span></h1>
              <p>Your destination for premium after-school learning.</p>
            </div>
          </section>

          <div class="section-header">
            <div class="heading-row">
              <h2>Available Courses</h2>
              <span class="course-badge">{{ products.length }} Courses</span>
            </div>
            <p class="heading-subtitle">
              Explore our wide range of after-school programs designed to inspire.
            </p>
          </div>

          <div v-if="isLoadingLessons" class="lessons">
            <div class="product-card skeleton-card" v-for="n in 6" :key="n">
              <div class="skeleton-image"></div>
              <div class="product-content">
                <div class="skeleton-text title"></div>
                <div class="skeleton-text subtitle"></div>
                <div class="skeleton-text desc"></div>
                <div class="skeleton-text btn"></div>
              </div>
            </div>
          </div>

          <p v-else-if="loadLessonsError" class="error-msg">
            {{ loadLessonsError }}
          </p>

          <ProductList
            v-else
            :products="filteredProducts"
            :cartCount="cartCount"
            :canAddToCart="canAddToCart"
            :spacesLeft="spacesLeft"
            @add-to-cart="addToCart"
            :sort-attribute="sortAttribute"
            :sort-order="sortOrder"
            @update-sort-attribute="val => (sortAttribute = val)"
            @update-sort-order="val => (sortOrder = val)"
            :search-term="searchTerm"
            @update-search-term="val => (searchTerm = val)"
          />
        </div>

        <CartView
          v-else
          :cart="cart"
          :cartProducts="cartProducts"
          :cartCount="cartCount"
          :order="order"
          :checkoutReady="checkoutReady"
          :isPlacingOrder="isPlacingOrder"             
          :orderSuccessMessage="orderSuccessMessage"  
          :orderErrorMessage="orderErrorMessage"
          :spacesLeft="spacesLeft" 
          @update-name="val => order.firstName = val"
          @update-phone="val => order.phone = val"
          @place-order="placeOrder"
          @remove-one="removeFromCart"
          @add-to-cart="addToCart"
          @toggle-checkout="toggleCheckout"
        />

      </Transition>

    </main>
  </div>
</template>

<script>
// --- IMPORTS ---
import ProductList from "./components/ProductList.vue";
import CartView from "./components/CartView.vue";
import CustomDropdown from "./components/CustomDropdown.vue";

// Central API base for the deployed backend
const API_BASE = "https://afterbellcorner-backend.onrender.com";

export default {
  name: "App",
  components: {
    ProductList,
    CartView,
    CustomDropdown
  },

  data() {
    return {
      sitename: "After Bell Corner",
      
      // UI State
      showProduct: true, // true = Browse, false = Checkout
      showFilters: false,
      isLoadingLessons: true,
      loadLessonsError: null,
      cartAnimating: false,
      scrollObserver: null,

      // Data
      products: [],
      cart: [], // Array of product IDs
      searchTerm: "",
      
      // Sorting Options
      sortAttribute: "subject",
      sortOrder: "asc",
      sortOptions: [
        { value: 'subject', label: 'Subject' },
        { value: 'location', label: 'Location' },
        { value: 'price', label: 'Price' },
        { value: 'spaces', label: 'Spaces' }
      ],
      orderOptions: [
        { value: 'asc', label: 'Ascending' },
        { value: 'desc', label: 'Descending' }
      ],

      // Checkout Form Data
      order: {
        firstName: "",
        phone: "",
      },
      isPlacingOrder: false,          
      orderSuccessMessage: "",        
      orderErrorMessage: "", 
    };
  },

  // --- LIFECYCLE HOOKS ---
  created() {
    // Initial fetch of lessons on load
    this.fetchLessons();
  },

  updated() {
    // Re-run observer whenever the DOM updates (e.g. view switch)
    // This is a backup check to ensure animations trigger if transition timing varies
    if (this.showProduct && !this.isLoadingLessons) {
      const cards = document.querySelectorAll('.product-card');
      if (cards.length > 0) {
        this.setupScrollObserver();
      }
    }
  },

  // --- COMPUTED PROPERTIES ---
  computed: {
    cartItemCount() {
      return this.cart.length || "";
    },

    // Returns unique product objects currently in the cart
    cartProducts() {
      const uniqueIds = [...new Set(this.cart)];
      return uniqueIds
        .map(id => this.products.find(p => p.id === id))
        .filter(Boolean);
    },

    // Sorts products based on selected attribute and order
    sortedProducts() {
      const arr = [...this.products];
      const attr = this.sortAttribute;
      const dir = this.sortOrder;

      arr.sort((a, b) => {
        let A, B;

        if (attr === "subject") {
          A = a.title; B = b.title;
        } else if (attr === "location") {
          A = a.location; B = b.location;
        } else if (attr === "price") {
          A = a.price; B = b.price;
        } else if (attr === "spaces") {
          A = a.availableInventory - this.cartCount(a.id);
          B = b.availableInventory - this.cartCount(b.id);
        } else {
          return 0;
        }

        if (typeof A === "string" && typeof B === "string") {
          A = A.toLowerCase(); B = B.toLowerCase();
          if (A < B) return dir === "asc" ? -1 : 1;
          if (A > B) return dir === "asc" ? 1 : -1;
          return 0;
        } else {
          if (A < B) return dir === "asc" ? -1 : 1;
          if (A > B) return dir === "asc" ? 1 : -1;
          return 0;
        }
      });
      return arr;
    },

    // Filters sorted products by search term
    filteredProducts() {
      const list = this.sortedProducts;
      if (!this.searchTerm) return list;

      const term = this.searchTerm.toLowerCase();
      return list.filter((product) => {
        const subject = product.title?.toLowerCase() || "";
        const location = product.location?.toLowerCase() || "";
        return subject.includes(term) || location.includes(term);
      });
    },

    // Validates form before enabling Place Order button
    checkoutReady() {
      const nameOk = /^[A-Za-z]+$/.test(this.order.firstName);
      const phoneOk = /^[0-9]+$/.test(String(this.order.phone));
      const hasItems = this.cart.length > 0;
      return nameOk && phoneOk && hasItems;
    },
  },

  // --- WATCHERS ---
  watch: {
    // Re-initialize animations when search results change
    searchTerm() {
      setTimeout(() => {
        this.setupScrollObserver();
      }, 100);
    },

    // Trigger badge animation when cart updates
    cart: {
      handler() {
        this.cartAnimating = true;
        setTimeout(() => {
          this.cartAnimating = false;
        }, 300);
      },
      deep: true
    }
  },

  // --- METHODS ---
  methods: {
    
    // Fetch lessons from the backend API
    async fetchLessons() {
      console.log("Fetching lessons from", `${API_BASE}/lessons`);
      this.isLoadingLessons = true;
      this.loadLessonsError = null;

      try {
        const response = await fetch(`${API_BASE}/lessons`);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        
        const data = await response.json();
        this.products = data;

        // Initialize scroll animations after data load
        setTimeout(() => {
          this.setupScrollObserver(); 
        }, 100);

      } catch (err) {
        console.error("Error loading lessons:", err);
        this.loadLessonsError = "Failed to load lessons from the server. Please try again later.";
      } finally {
        this.isLoadingLessons = false;
      }
    },

    // Sets up the IntersectionObserver for scroll-reveal animations
    setupScrollObserver() {
      if (this.scrollObserver) this.scrollObserver.disconnect();

      const cards = document.querySelectorAll('.product-card');
      if (cards.length === 0) return;

      this.scrollObserver = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('in-view');
          } else {
            entry.target.classList.remove('in-view');
          }
        });
      }, { threshold: 0.1 });

      cards.forEach((card) => this.scrollObserver.observe(card));
    },

    // Called when the "Back to Home" transition finishes
    onAfterEnter() {
      if (this.showProduct) {
        this.$nextTick(() => {
          this.setupScrollObserver();
        });
      }
    },

    // View Navigation
    toggleCheckout() {
      this.showProduct = !this.showProduct;
      // Reset messages when switching views
      this.orderSuccessMessage = "";
      this.orderErrorMessage = "";
    },

    // Cart Logic
    cartCount(id) {
      let count = 0;
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === id) count++;
      }
      return count;
    },

    spacesLeft(product) {
      const left = product.availableInventory - this.cartCount(product.id);
      return left < 0 ? 0 : left;
    },

    canAddToCart(product) {
      return this.spacesLeft(product) > 0;
    },

    addToCart(product) {
      if (this.canAddToCart(product)) {
        this.cart.push(product.id);
      }
    },

    removeFromCart(product) {
      const idx = this.cart.indexOf(product.id);
      if (idx !== -1) {
        this.cart.splice(idx, 1);
      }
    },

    // Checkout Logic
    async placeOrder() {
      if (!this.checkoutReady) return;

      const name = this.order.firstName;     
      const phone = this.order.phone;
      const items = this.cartProducts.map((product) => ({
        id: product.id,
        qty: this.cartCount(product.id),
      }));

      this.isPlacingOrder = true;          
      this.orderSuccessMessage = "";
      this.orderErrorMessage = "";

      try {
        const response = await fetch(`${API_BASE}/orders`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ name, phone, items }),
        });

        if (!response.ok) {
          let errorText = `HTTP ${response.status}`;
          try {
            const body = await response.json();
            if (body && body.error) errorText = body.error;
          } catch (_) {}
          throw new Error(errorText);
        }

        const data = await response.json();
        this.orderSuccessMessage = `Order placed successfully. Your reference ID is ${data.orderId}.`;

        // Auto-dismiss success message
        setTimeout(() => {
          this.orderSuccessMessage = "";
        }, 4000);

        await this.updateLessonSpacesAfterOrder(items);

        // Reset Cart
        this.cart = [];
        this.order.firstName = "";
        this.order.phone = "";

      } catch (err) {
        console.error("Error placing order:", err);
        this.orderErrorMessage = "Sorry, there was a problem placing your order. Please try again.";
      } finally {
        this.isPlacingOrder = false;      
      }
    },

    // Sync inventory with backend
    async updateLessonSpacesAfterOrder(items) {
      for (const item of items) {
        const lesson = this.products.find(p => p.id === item.id);
        if (!lesson) continue;

        const newAvailable = lesson.availableInventory - item.qty;
        const safeAvailable = newAvailable < 0 ? 0 : newAvailable;

        try {
          await fetch(`${API_BASE}/lessons/${item.id}`, {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ availableInventory: safeAvailable }),
          });
          lesson.availableInventory = safeAvailable;
        } catch (err) {
          console.error(`Error updating spaces for lesson ${item.id}:`, err);
        }
      }
    }
  },

  // --- CUSTOM DIRECTIVES ---
  directives: {
    'click-outside': {
      mounted(el, binding) {
        el.clickOutsideEvent = function(event) {
          if (!(el === event.target || el.contains(event.target))) {
            const toggleBtn = document.querySelector('.filter-btn');
            if (toggleBtn && toggleBtn.contains(event.target)) return;
            binding.value(event);
          }
        };
        document.body.addEventListener('click', el.clickOutsideEvent);
      },
      unmounted(el) {
        document.body.removeEventListener('click', el.clickOutsideEvent);
      },
    },
  }
};
</script>