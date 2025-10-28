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

      <CartView
        v-else
        :cart="cart"
        :cartProducts="cartProducts"
        :cartCount="cartCount"
        :order="order"
        :checkoutReady="checkoutReady"
        @update-name="val => order.firstName = val"
        @update-phone="val => order.phone = val"
        @place-order="placeOrder"
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

      // showProduct === true  -> user is browsing sessions
      // showProduct === false -> user is in the cart / checkout screen
      showProduct: true,

      // cart is just an array of product IDs.
      // Quantity of an item = how many times that ID appears.
      cart: [],
      
      // Hard-coded "lessons" / "products" data.
      // availableInventory is used to limit how many can be added.
      // rating is used to render ★★★★☆ in ProductList.
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
        }
      ],
      
      // sorting controls for ProductList.
      // sortAttribute is which field we sort by (price/title/availableInventory)
      // sortOrder is 'asc' or 'desc'
      sortAttribute: "price",
      sortOrder: "asc",
      
      // order = checkout form data.
      // firstName must be letters only.
      // phone must be digits only.
      order: {
        firstName: "",
        phone: "",
      },
    };
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

    // sortedProducts returns the products array sorted
    // by whatever dropdown is chosen in ProductList
    // (price/title/availableInventory + asc/desc).
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
    canAddToCart(product) {
      return product.availableInventory > this.cartCount(product.id);
    },

    // Called when "Add to Cart" is clicked in ProductList
    addToCart(product) {
      if (this.canAddToCart(product)) {
        this.cart.push(product.id);
      }
    },

    // placeOrder is passed to CartView and runs when the
    // user clicks "Place Order". It also resets the app.
    placeOrder() {
      if (!this.checkoutReady) return;

      alert("Order placed! 🎉");

      // clear form data
      this.order.firstName = "";
      this.order.phone = "";

      // clear all items in cart
      this.cart = [];

      // jump back to product browsing
      this.showProduct = true;
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

</style>
