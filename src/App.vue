<template>
  <div class="min-h-screen bg-gray-100">
    <!-- Header -->
    <header class="bg-white shadow-md">
      <div class="container mx-auto px-4 py-6 flex justify-between items-center">
        <div class="text-2xl font-bold text-gray-800">ClothingCo</div>
        <nav>
          <ul class="flex space-x-4">
            <li><a href="#" class="text-gray-600 hover:text-gray-800">Home</a></li>
            <li><a href="#" class="text-gray-600 hover:text-gray-800">Categories</a></li>
            <li><a href="#" class="text-gray-600 hover:text-gray-800">About</a></li>
            <li><a href="#" class="text-gray-600 hover:text-gray-800">Contact</a></li>
          </ul>
        </nav>
      </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto px-4 py-8">
      <h1 class="text-4xl font-bold text-center mb-8">Find Your Perfect Style</h1>
      
      <!-- Search Bar -->
      <div class="max-w-2xl mx-auto mb-8">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search for clothing..."
          class="w-full px-4 py-2 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          @input="handleSearch"
        />
      </div>

      <!-- Loading Indicator -->
      <div v-if="isLoading" class="text-center">
        <div class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-blue-500"></div>
        <p class="mt-2 text-gray-600">Loading...</p>
      </div>

      <!-- Search Results -->
      <div v-else-if="searchQuery.length > 3" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <transition-group name="fade">
          <div
            v-for="item in filteredProducts"
            :key="item.id"
            class="bg-white rounded-lg shadow-md overflow-hidden transition-transform duration-300 ease-in-out transform hover:scale-105"
          >
            <img :src="item.image" :alt="item.title" class="w-full h-64 object-contain p-4" />
            <div class="p-4">
              <h2 class="text-xl font-semibold mb-2">{{ item.title }}</h2>
              <p class="text-gray-600 mb-4">{{ item.description }}</p>
              <div class="flex justify-between items-center">
                <span class="text-2xl font-bold text-blue-600">${{ item.price.toFixed(2) }}</span>
                <button class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 transition-colors duration-300">
                  Add to Cart
                </button>
              </div>
            </div>
          </div>
        </transition-group>
      </div>

      <!-- No Results Message -->
      <div v-else-if="searchQuery.length > 3 && filteredProducts.length === 0" class="text-center text-gray-600">
        No results found for "{{ searchQuery }}". Try a different search term.
      </div>

      <!-- Initial Message -->
      <div v-else class="text-center text-gray-600">
        <p> Start typing words like Jacket,Short,T shirt,Bagpack etc, to search for clothing items...</p> 
        <p> Data fetched from <a class="text-blue-500" :href="Link">fakestoreapi</a> </p> 
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const searchQuery = ref('')
const products = ref([])
const isLoading = ref(false)
const Link='https://fakestoreapi.com'
const fetchProducts = async () => {
  isLoading.value = true
  try {
    const response = await fetch('https://fakestoreapi.com/products')
    const data = await response.json()
    products.value = data.filter(product => 
      product.category === "men's clothing" || product.category === "women's clothing"
    )
  } catch (error) {
    console.error('Error fetching products:', error)
  } finally {
    isLoading.value = false
  }
}

onMounted(fetchProducts)

const filteredProducts = computed(() => {
  if (searchQuery.value.length <= 3) return []
  return products.value.filter(product =>
    product.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    product.description.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const handleSearch = () => {
  // The search is now handled by the computed property
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>