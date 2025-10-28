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
      
        <ProductList
          v-if="showProduct"
          :products="filteredProducts"
          :cartCount="cartCount"
          :canAddToCart="canAddToCart"
          :spacesLeft="spacesLeft"
          @add-to-cart="addToCart"

          :sort-attribute="sortAttribute"
          :sort-order="sortOrder"
          @update-sort-attribute="val => sortAttribute = val"
          @update-sort-order="val => sortOrder = val"

          :search-term="searchTerm"
          @update-search-term="val => searchTerm = val"
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
        description: "Focused small-group support for core exam skills.",
        price: 20,
        availableInventory: 5,
        rating: 4,
        location: "Room 101",
      },
      {
        id: 1002,
        title: "Science Lab Club",
        description: "Hands-on experiments with safety guidance.",
        price: 25,
        availableInventory: 5,
        rating: 5,
        location: "Lab 2",
      },
      {
        id: 1003,
        title: "Creative Writing Workshop",
        description: "Storycraft, characters, and confidence building.",
        price: 15,
        availableInventory: 5,
        rating: 3,
        location: "Library",
      },
      {
        id: 1004,
        title: "After-School Coding Club",
        description: "Intro to programming with puzzles and games.",
        price: 18,
        availableInventory: 5,
        rating: 4,
        location: "ICT Suite",
      },
      {
        id: 1005,
        title: "Art & Design Studio",
        description: "Painting, sketching and creative techniques.",
        price: 22,
        availableInventory: 5,
        rating: 5,
        location: "Art Room",
      },
      {
        id: 1006,
        title: "History Revision Clinic",
        description: "Key events, timelines and exam tips.",
        price: 17,
        availableInventory: 5,
        rating: 4,
        location: "Room 204",
      },
      {
        id: 1007,
        title: "Geography Field Skills",
        description: "Maps, fieldwork techniques and case studies.",
        price: 19,
        availableInventory: 5,
        rating: 4,
        location: "Geography Room",
      },
      {
        id: 1008,
        title: "English Literature Circle",
        description: "Discussion of key texts and themes.",
        price: 16,
        availableInventory: 5,
        rating: 3,
        location: "Room 110",
      },
      {
        id: 1009,
        title: "Music Practice Session",
        description: "Instrumental practice with guidance.",
        price: 21,
        availableInventory: 5,
        rating: 5,
        location: "Music Room",
      },
      {
        id: 1010,
        title: "Exam Stress Toolkit",
        description: "Study skills, organisation and wellbeing.",
        price: 14,
        availableInventory: 5,
        rating: 4,
        location: "Wellbeing Hub",
      },
    ],

      
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
