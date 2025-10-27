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

    <hr />

    <h3>Checkout Details</h3>

    <p>
      <strong>Name:</strong>
      <input
        :value="order.firstName"
        @input="$emit('update-name', $event.target.value.trim())"
        placeholder="First name only (letters)"
      />
    </p>

    <p>
      <strong>Phone:</strong>
      <input
        :value="order.phone"
        @input="$emit('update-phone', $event.target.value.replace(/[^0-9]/g,''))"
        placeholder="Numbers only"
      />
    </p>

    
    <button
      @click="$emit('place-order')"
      :disabled="!checkoutReady"
    >
      Place Order
    </button>

    <p class="back-hint">
      <em>Click "Checkout" again to go back to products.</em>
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
    order: Object,          
    checkoutReady: Boolean,
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
