<template>
  <div id="app">
    <header>
      <h1>{{ sitename }}</h1>
      
      <button disabled>
        0
        <span class="fas fa-cart-plus"></span>
        Checkout
      </button>
    </header>

    <main>
      <p>App running...</p>
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      
      sitename: "After Bell Corner",
    };
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
}
</style>
