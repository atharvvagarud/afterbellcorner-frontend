<template>
  <div class="custom-dropdown" ref="dropdownRef">
    
    <div class="dropdown-trigger" @click="toggleDropdown">
      <span>{{ selectedLabel }}</span>
      <span class="fas fa-chevron-down chevron" :class="{ 'rotate': isOpen }"></span>
    </div>

    <Transition name="zoom">
      <ul v-if="isOpen" class="dropdown-menu">
        <li
          v-for="option in options"
          :key="option.value"
          class="dropdown-item"
          :class="{ 'active': modelValue === option.value }"
          @click="selectOption(option.value)"
        >
          <span>{{ option.label }}</span>
          <span v-if="modelValue === option.value" class="fas fa-check check-icon"></span>
        </li>
      </ul>
    </Transition>

  </div>
</template>

<script>
export default {
  name: 'CustomDropdown',
  props: {
    modelValue: String, 
    options: Array     
  },
  data() {
    return {
      isOpen: false
    }
  },
  computed: {
    selectedLabel() {
      const found = this.options.find(o => o.value === this.modelValue);
      return found ? found.label : this.modelValue;
    }
  },
  mounted() {
    // Close dropdown when clicking anywhere else on the page
    document.addEventListener('click', this.handleClickOutside);
  },
  unmounted() {
    document.removeEventListener('click', this.handleClickOutside);
  },
  methods: {
    toggleDropdown() {
      this.isOpen = !this.isOpen;
    },
    selectOption(value) {
      this.$emit('update:modelValue', value);
      this.isOpen = false;
    },
    handleClickOutside(event) {
      if (this.$refs.dropdownRef && !this.$refs.dropdownRef.contains(event.target)) {
        this.isOpen = false;
      }
    }
  }
}
</script>