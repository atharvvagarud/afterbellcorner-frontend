<template>
  <div id="app">
    <header>
      <h1>{{ sitename }}</h1>
      
      <button @click="toggleCheckout">
        {{ cartItemCount }}
        <span class="fas fa-cart-plus"></span>
        Checkout
      </button>
    </header>

    <main>
      
        <ProductList
        v-if="showProduct"
        :products="sortedProducts"
        :cartCount="cartCount"
        :canAddToCart="canAddToCart"
        @add-to-cart="addToCart"

        :sort-attribute="sortAttribute"
        :sort-order="sortOrder"
        @update-sort-attribute="val => sortAttribute = val"
        @update-sort-order="val => sortOrder = val"
      />

      <div v-else>
        <h2>Cart / Checkout (placeholder)</h2>
        <p>render cart here later</p>
      </div>

    </main>
  </div>
</template>

<script>
import ProductList from "./components/ProductList.vue";

export default {
  name: "App",
  components: {
    ProductList
  },

  data() {
    return {
      
      sitename: "After Bell Corner",

      showProduct: true,

      cart: [],

      products: [
        {
          id: 1001,
          title: "Maths Booster Session",
          description: "Focused small-group support for exam topics.",
          price: 20,
          availableInventory: 5,
          rating: 4,
        },
        {
          id: 1002,
          title: "Science Lab Club",
          description: "Hands-on experiments and safety basics.",
          price: 25,
          availableInventory: 3,
          rating: 5,
        },
        {
          id: 1003,
          title: "Creative Writing Workshop",
          description: "Storycraft, characters, and confidence.",
          price: 15,
          availableInventory: 10,
          rating: 3,
        },
      
      ],
      sortAttribute: "price",  
      sortOrder: "asc",        
    };
  },
  computed: {
    cartItemCount() {
      return this.cart.length || "";
    },
    sortedProducts() {
      const arr = [...this.products];
      const attr = this.sortAttribute;
      const dir = this.sortOrder;

      arr.sort((a, b) => {
        let A = a[attr];
        let B = b[attr];

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
    }
  },
  methods: {
    toggleCheckout() {
      this.showProduct = !this.showProduct;
    },
    cartCount(id) {
      let count = 0;
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === id) {
          count++;
        }
      }
      return count;
    },
    canAddToCart(product) {
      return product.availableInventory > this.cartCount(product.id);
    },
    addToCart(product) {
      if (this.canAddToCart(product)) {
        this.cart.push(product.id);
      }
    }
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


</style>
