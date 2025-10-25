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
      
      <div v-if="showProduct">
        <h2>Available Sessions (placeholder)</h2>
        <p>render lessons here later</p>
      </div>

      <div v-else>
        <h2>Cart / Checkout (placeholder)</h2>
        <p>render cart here later</p>
      </div>

    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      
      sitename: "After Bell Corner",

      showProduct: true,

      cart: [],
    };
  },
  computed: {
    cartItemCount() {
      return this.cart.length || "";
    },
  },
  methods: {
    toggleCheckout() {
      this.showProduct = !this.showProduct;
    },
  },
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
