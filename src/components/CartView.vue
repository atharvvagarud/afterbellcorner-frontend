<template>
  <div class="cart-wrapper">
    
    <div class="cart-section">
      <h2 class="section-title">Your Cart ({{ cart.length }})</h2>

      <div 
        v-if="cart.length === 0" 
        class="empty-cart-msg"
      >
        <i class="fas fa-shopping-basket fa-3x"></i>
        <p>Your cart is empty.</p>
        <button @click="$emit('toggle-checkout')" class="secondary-btn">
          Go back to Lessons
        </button>
      </div>

      <TransitionGroup 
        v-else
        name="list" 
        tag="div" 
        class="cart-list" 
      >
        <div
          class="cart-item"
          v-for="product in cartProducts"
          :key="product.id"
        >
          <div class="item-details">
            <span class="item-title">{{ product.title }}</span>
            <span class="item-price">£{{ product.price }}</span>
          </div>

          <div class="item-actions stepper">
            <button 
              @click="$emit('remove-one', product)" 
              class="step-btn minus"
              title="Decrease quantity"
            >
              <i class="fas fa-minus"></i>
            </button>
            
            <span class="qty-display">{{ cartCount(product.id) }}</span>
            
            <button 
              @click="$emit('add-to-cart', product)" 
              class="step-btn plus"
              title="Increase quantity"
              :disabled="spacesLeft(product) === 0"
            >
              <i class="fas fa-plus"></i>
            </button>
          </div>
        </div>
      </TransitionGroup>
      
    </div>

    <div class="checkout-section">
      <div class="checkout-card">
        <h3>Checkout Details</h3>

        <div v-if="orderSuccessMessage" class="msg success">
          <i class="fas fa-check-circle"></i> {{ orderSuccessMessage }}
        </div>
        <div v-if="orderErrorMessage" class="msg error">
          <i class="fas fa-exclamation-triangle"></i> {{ orderErrorMessage }}
        </div>

        <form @submit.prevent="$emit('place-order')">
          
          <div class="form-group">
            <label>Name</label>
            <div class="input-wrapper">
              <span class="fas fa-user input-icon"></span>
              <input
                :value="order.firstName"
                @input="$emit('update-name', $event.target.value.trim())"
                placeholder="Enter your name"
                required
                pattern="[A-Za-z]+" 
                title="Name must contain letters only"
              />
            </div>
          </div>

          <div class="form-group">
            <label>Phone</label>
            <div class="input-wrapper">
              <span class="fas fa-phone input-icon"></span>
              <input
                :value="order.phone"
                @input="$emit('update-phone', $event.target.value.replace(/[^0-9]/g,''))"
                placeholder="Enter phone number"
                required
                pattern="[0-9]{7,}" 
                title="Phone must be at least 7 digits"
              />
            </div>
          </div>

          <div class="order-summary">
            <div class="summary-row">
              <span>Total Items:</span>
              <span>{{ cart.length }}</span>
            </div>

            <div class="summary-row total">
              <span>Total To Pay:</span>
              <span :key="totalPrice" class="total-price-text">
                £{{ totalPrice }}
              </span>
            </div>
          </div>

          <button
            type="submit"
            :disabled="!checkoutReady || isPlacingOrder"
            class="checkout-btn"
          >
            {{ isPlacingOrder ? "Processing..." : "Place Order" }}
          </button>
        </form>
        
        <p class="back-hint">
            Review your items before placing the order.
        </p>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: "CartView",
  props: {
    cart: Array,
    cartProducts: Array,
    cartCount: Function,
    spacesLeft: Function, 
    order: Object,          
    checkoutReady: Boolean,
    isPlacingOrder: Boolean,
    orderSuccessMessage: String,
    orderErrorMessage: String,
  },
  computed: {
    totalPrice() {
      return this.cart.reduce((total, id) => {
        const product = this.cartProducts.find(p => p.id === id);
        return total + (product ? product.price : 0);
      }, 0);
    }
  }
};
</script>
