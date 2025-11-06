<template>
  <div id="app">
    <header>
      <h1>{{ sitename }}</h1>
      
  <button
    @click="toggleCheckout"
    :disabled="cart.length === 0 && showProduct"
    :title="cart.length === 0 && showProduct
    ? 'Add at least one session to view your cart'
    : 'View cart / checkout'"
  >
    {{ cartItemCount }}
    <span class="fas fa-cart-plus"></span>
    Checkout
  </button>
    </header>

    <main>
      
       <!-- LESSONS / PRODUCT LIST VIEW -->
  <div v-if="showProduct">

        <p v-if="isLoadingLessons" class="loading-msg">
          Loading lessons...
        </p>

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
        @update-name="val => order.firstName = val"
        @update-phone="val => order.phone = val"
        @place-order="placeOrder"
        @remove-one="removeFromCart"
      />

    </main>
  </div>
</template>

<script>
import ProductList from "./components/ProductList.vue";
import CartView from "./components/CartView.vue";

export default {
  name: "App",
  components: {
    ProductList,
    CartView
  },

  data() {
    return {
      sitename: "After Bell Corner",

      products: [],

      // simple loading + error state for lessons fetch
      isLoadingLessons: true,
      loadLessonsError: null,

      // showProduct === true  -> user is browsing sessions
      // showProduct === false -> user is in the cart / checkout screen
      showProduct: true,

      // cart is just an array of product IDs.
      // Quantity of an item = how many times that ID appears.
      cart: [],
      
      // sorting controls for ProductList.
      // sortAttribute is which field we sort by (price/title/availableInventory)
      // sortOrder is 'asc' or 'desc'
      sortAttribute: "subject",
      sortOrder: "asc",

      searchTerm: "",
      
      // order = checkout form data.
      // firstName must be letters only.
      // phone must be digits only.
      order: {
        firstName: "",
        phone: "",
      },

      //lags + messages for checkout
      isPlacingOrder: false,          
      orderSuccessMessage: "",        
      orderErrorMessage: "", 
      
    };
  },

  created() {
    //call the loader as soon as the app is created
    this.fetchLessons()
  },

  computed: {
    cartItemCount() {
      return this.cart.length || "";
    },
     cartProducts() {
      const uniqueIds = [...new Set(this.cart)];
      return uniqueIds
        .map(id => this.products.find(p => p.id === id))
        .filter(Boolean);
    },


sortedProducts() {
  const arr = [...this.products];
  const attr = this.sortAttribute;
  const dir = this.sortOrder;

  arr.sort((a, b) => {
    let A;
    let B;

    if (attr === "subject") {
      A = a.title;
      B = b.title;
    } else if (attr === "location") {
      A = a.location;
      B = b.location;
    } else if (attr === "price") {
      A = a.price;
      B = b.price;
    } else if (attr === "spaces") {
     
      A = a.availableInventory - this.cartCount(a.id);
      B = b.availableInventory - this.cartCount(b.id);
    } else {
      
      return 0;
    }

    
    if (typeof A === "string" && typeof B === "string") {
      A = A.toLowerCase();
      B = B.toLowerCase();
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

filteredProducts() {
  // start from the sorted list
  const list = this.sortedProducts;

  // if search box is empty, show everything
  if (!this.searchTerm) {
    return list;
  }

  const term = this.searchTerm.toLowerCase();

  // filter by subject (title) OR location
  return list.filter((product) => {
    const subject = product.title?.toLowerCase() || "";
    const location = product.location?.toLowerCase() || "";
    return (
      subject.includes(term) ||
      location.includes(term)
    );
  });
},

    // The "Place Order" button is only enabled if:
    //   - firstName is only letters
    //   - phone is only numbers
    //   - there is at least one item in the cart
    checkoutReady() {
      const nameOk = /^[A-Za-z]+$/.test(this.order.firstName);
      const phoneOk = /^[0-9]+$/.test(String(this.order.phone));
      const hasItems = this.cart.length > 0;
      return nameOk && phoneOk && hasItems;
    },
  },
  
methods: {

    async fetchLessons() {
    this.isLoadingLessons = true;
    this.loadLessonsError = null;

    try {
      const response = await fetch("http://localhost:3000/lessons");
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      const data = await response.json();
      this.products = data;
    } catch (err) {
      console.error("Error loading lessons:", err);
      this.loadLessonsError =
        "Failed to load lessons from the server. Please try again later.";
    } finally {
      this.isLoadingLessons = false;
    }
    },

    // Toggle between product browsing vs. checkout view
    toggleCheckout() {
      this.showProduct = !this.showProduct;
    },

    // Returns how many of a given product ID are currently in cart
    cartCount(id) {
      let count = 0;
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === id) {
          count++;
        }
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

    // Called when "Add to Cart" is clicked in ProductList
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

    // Called when "Place Order" is clicked in CartView
    async placeOrder() {
    
    if (!this.checkoutReady) {
      return;
    }

    
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
      const response = await fetch("http://localhost:3000/orders", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          name,
          phone,
          items,
        }),
      });

      if (!response.ok) {
        
        let errorText = `HTTP ${response.status}`;
        try {
          const body = await response.json();
          if (body && body.error) {
            errorText = body.error;
          }
        } catch (_) {}

        throw new Error(errorText);
      }

      const data = await response.json();

      
      this.orderSuccessMessage = `Order placed successfully. Your reference ID is ${data.orderId}.`;

      // clear cart + order form
      this.cart = [];
      this.order.firstName = "";
      this.order.phone = "";

      // go back to product view
      this.showProduct = true;
    } catch (err) {
      console.error("Error placing order:", err);
      this.orderErrorMessage =
        "Sorry, there was a problem placing your order. Please try again.";
    } finally {
      this.isPlacingOrder = false;      
    }
  },
  }
};
</script>

<style scoped>

header {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid #ccc;
  padding-bottom: 1rem;
  margin-bottom: 1.5rem;
}

header h1 {
  margin: 0;
  font-size: 1.2rem;
}

header button {
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  cursor: pointer;
}

.loading-msg {
  text-align: center;
  margin: 1rem 0;
  font-style: italic;
}

.error-msg {
  text-align: center;
  margin: 1rem 0;
  color: #c0392b;
  font-weight: bold;
}

</style>
