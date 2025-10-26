<template>
  <div>
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

.in-cart-label {
  font-size: 0.8rem;
  margin-top: 0.5rem;
}
</style>
