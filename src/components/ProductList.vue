<template>
  <div>


<!-- TOOLBAR: search + sort -->

<div class="toolbar">

  <!-- Search pill -->
  <div class="search-box">
    <span class="search-icon">🔍</span>

    <input
      class="search-input"
      type="text"
      :value="searchTerm"
      @input="$emit('update-search-term', $event.target.value)"
      placeholder="Search by subject or location"
    />
  </div>

  <!-- Sort controls -->
  <div class="sort-group">
    <span class="sort-label">Sort</span>

    <select
      class="sort-select"
      :value="sortAttribute"
      @change="$emit('update-sort-attribute', $event.target.value)"
    >
      <option value="subject">Subject</option>
      <option value="location">Location</option>
      <option value="price">Price</option>
      <option value="spaces">Spaces</option>
    </select>

    <select
      class="sort-select"
      :value="sortOrder"
      @change="$emit('update-sort-order', $event.target.value)"
    >
      <option value="asc">Ascending</option>
      <option value="desc">Descending</option>
    </select>
  </div>

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

          <!-- Subject + Icon -->
          <h2>
              <span class="fas fa-chalkboard-teacher lesson-icon"></span>
              {{ product.title }}
          </h2>

          <p>{{ product.description }}</p>


          <p><strong>Location:</strong> {{ product.location }}</p>
          <p><strong>Price:</strong> £{{ product.price }}</p>
          <p> <strong>Spaces:</strong> {{ spacesLeft(product) }} </p>


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
              
            <span v-if="spacesLeft(product) === 0">
               All out!
            </span>

            <span v-else-if="spacesLeft(product) < 5">
              Only
              {{ spacesLeft(product) }}
              left!
            </span>

            <span v-else>
              Buy now!
            </span>
          </p>

         <button
            v-if="canAddToCart(product)"
            @click="$emit('add-to-cart', product)"
            aria-label="Add {{ product.title }} to cart"
          >
            Add to Cart
          </button>

          <button
            v-else
            disabled="disabled"
            title="No spaces left for this session"
            aria-label="No spaces left for {{ product.title }}"
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
    spacesLeft: Function,
    sortAttribute: String,
    sortOrder: String,
    searchTerm: String,
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

.lesson-icon {
  margin-right: 0.4rem;
}

.search-row {
  margin-bottom: 0.75rem;
  font-size: 0.9rem;
}

.search-row label {
  margin-right: 0.5rem;
  font-weight: bold;
}

.search-row input {
  padding: 0.2rem 0.4rem;
  font-size: 0.9rem;
}

button {
  transition: all 0.2s ease-in-out;
}

button:not(:disabled):hover {
  background-color: #333;
  color: white;
  transform: scale(1.03);
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.empty-cart-msg {
  text-align: center;
  margin: 1rem 0;
  font-style: italic;
  color: #666;
}

</style>
