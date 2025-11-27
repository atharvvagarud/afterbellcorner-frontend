<template>
  <div>
    <section class="lessons">
  <article
  v-for="product in products"
  :key="product.id"
  class="product-card group"
>
  <div class="product-image">
    <img 
      :src="getImagePath(product.title)" 
      :alt="product.title"
      class="card-img"
    />
    <div v-if="spacesLeft(product) === 0" class="sold-out-overlay">
      <span>Sold Out</span>
    </div>
  </div>

  <div class="product-content">
    
    <div class="card-header">
      <h2 class="card-title">{{ product.title }}</h2>
      <span class="card-price">£{{ product.price }}</span>
    </div>

    <div class="card-meta">
      <span class="location">
        <i class="fas fa-map-marker-alt"></i> {{ product.location }}
      </span>
      <div class="rating">
        <span v-for="n in product.rating" :key="n">★</span>
        <span v-for="n in 5 - product.rating" :key="n" class="text-muted">☆</span>
      </div>
    </div>

    <p class="card-desc">{{ product.description }}</p>

    <div class="card-footer">
      <span 
        v-if="spacesLeft(product) > 0"
        class="inventory-badge" 
        :class="spacesLeft(product) < 5 ? 'low-stock' : 'in-stock'"
      >
         {{ spacesLeft(product) < 5 ? `Only ${spacesLeft(product)} left!` : 'Available' }}
      </span>
      <span v-else class="h-6 block"></span> <button
        v-if="canAddToCart(product)"
        @click="$emit('add-to-cart', product)"
        class="add-btn"
      >
        Add to Cart
        <span v-if="cartCount(product.id) > 0" class="cart-count-badge">
          ({{ cartCount(product.id) }})
        </span>
      </button>

      <button
        v-else
        disabled
        class="add-btn disabled"
      >
        Sold Out
      </button>
    </div>
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
  methods: {
    getImagePath(title) {
      if (!title) return '/images/math.jpg'; // Safety fallback
      
      const t = title.toLowerCase();
      let filename = 'math.jpg'; // Default image

      if (t.includes('math')) filename = 'math.jpg';
      else if (t.includes('science')) filename = 'science.jpg';
      else if (t.includes('writing')) filename = 'writing.jpg';
      else if (t.includes('coding')) filename = 'coding.jpg';
      else if (t.includes('art')) filename = 'art.jpg';
      else if (t.includes('history')) filename = 'history.jpg';
      else if (t.includes('geo')) filename = 'geo.jpg';
      else if (t.includes('english')) filename = 'english.jpg';
      else if (t.includes('music')) filename = 'music.jpg';
      else if (t.includes('stress') || t.includes('wellbeing')) filename = 'wellbeing.jpg';

      // Assumes images are in the "public/images" folder
      return `images/${filename}`; 
    }
  }
};
</script>
