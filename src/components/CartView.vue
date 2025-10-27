<template>
  <section>
    <h2>Your Cart</h2>

    
    <div
      class="cart-list-item"
      v-for="product in cartProducts"
      :key="product.id"
    >
      <div>
        <strong>{{ product.title }}</strong>
        <div class="cart-price">
          £{{ product.price }} each
        </div>
      </div>

      <div class="cart-qty">
        Qty: {{ cartCount(product.id) }}
      </div>
    </div>

    <p v-if="cart.length === 0">
      Your cart is empty.
    </p>
  </section>
</template>

<script>
export default {
  name: "CartView",
  props: {
    cart: Array,
    cartProducts: Array,
    cartCount: Function,
  },
};
</script>

<style scoped>
.cart-list-item {
  border-bottom: 1px solid #ddd;
  padding: 0.5rem 0;
  display: flex;
  justify-content: space-between;
}

.cart-price {
  font-size: 0.8rem;
}

.cart-qty {
  text-align: right;
}
</style>
