<template>
  <div>

    <div class="sort-row">
      <label for="sortAttribute">Sort by:</label>
      <select
        id="sortAttribute"
        :value="sortAttribute"
        @change="$emit('update-sort-attribute', $event.target.value)"
      >
        <option value="price">Price</option>
        <option value="title">Title</option>
        <option value="availableInventory">Availability</option>
      </select>

      <label class="sort-order-label" for="sortOrder">Order:</label>
      <select
        id="sortOrder"
        :value="sortOrder"
        @change="$emit('update-sort-order', $event.target.value)"
      >
        <option value="asc">Ascending</option>
        <option value="desc">Descending</option>
      </select>
    </div>

    <section class="lessons">
      <article
        v-for="product in products"
        :key="product.id"
        class="product-card"
      >
        <div class="product-image">
          IMG {{ product.id }}
        </div>

        <div class="product-info">
          <h2>{{ product.title }}</h2>
          <p>{{ product.description }}</p>
          <p>Price: £{{ product.price }}</p>

           <div class="rating">
            <span
              v-for="n in product.rating"
              :key="product.id + '-full-' + n"
            >
              ★
            </span>
            <span
              v-for="n in 5 - product.rating"
              :key="product.id + '-empty-' + n"
            >
              ☆
            </span>
          </div>

           <p class="inventory-msg">
            <span v-if="product.availableInventory === cartCount(product.id)">
              All out!
            </span>

            <span
              v-else-if="product.availableInventory - cartCount(product.id) < 5"
            >
              Only
              {{ product.availableInventory - cartCount(product.id) }}
              left!
            </span>

            <span v-else>
              Buy now!
            </span>
          </p>

          <!-- Button respects inventory -->
          <button
            :disabled="!canAddToCart(product)"
            @click="$emit('add-to-cart', product)"
          >
            Add to Cart
          </button>

          <p class="in-cart-label">
            In cart: {{ cartCount(product.id) }}
          </p>
        </div>
      </article>
    </section>
  </div>
</template>

<script>
export default {
  name: "ProductList",
  props: {
    
    products: Array,
    cartCount: Function,
    canAddToCart: Function,

    sortAttribute: String,
    sortOrder: String,
  },
};
</script>

<style scoped>
/* Component local styling */

.product-card {
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 1rem;
  margin-bottom: 1rem;
  display: flex;
  gap: 1rem;
}

.product-image {
  width: 120px;
  height: 120px;
  background: #f3f3f3;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  color: #555;
}

.product-info h2 {
  margin: 0 0 0.5rem 0;
  font-size: 1.1rem;
}

.rating {
  color: #ffbf00;
  font-size: 0.9rem;
  line-height: 1;
  margin-bottom: 0.5rem;
}

.inventory-msg {
  font-size: 0.8rem;
  font-weight: bold;
  margin-top: 0.5rem;
}


.in-cart-label {
  font-size: 0.8rem;
  margin-top: 0.5rem;
}
</style>
