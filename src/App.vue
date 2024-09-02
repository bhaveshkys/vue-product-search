<template>
  <div class="min-h-screen bg-gray-100">
  
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

    
    <main class="container mx-auto px-4 py-8">
      <h1 class="text-4xl font-bold text-center mb-8">Find Your Perfect Style</h1>
      
      <div class="max-w-2xl mx-auto mb-8">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search for clothing..."
          class="w-full px-4 py-2 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          @input="handleSearch"
        />
      </div>

      <div v-if="isLoading" class="text-center">
        <div class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-blue-500"></div>
        <p class="mt-2 text-gray-600">Loading...</p>
      </div>

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

      <div v-else-if="searchQuery.length > 3 && filteredProducts.length === 0" class="text-center text-gray-600">
        No results found for "{{ searchQuery }}". Try a different search term.
      </div>

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
  if (searchQuery.value.length <= 3) return [];

  const threshold = 40; 
  
  return products.value.filter(product => {
    const titleDistance = levenshteinDistance(
      product.title.toLowerCase(),
      searchQuery.value.toLowerCase()
    );

    const descriptionDistance = levenshteinDistance(
      product.description.toLowerCase(),
      searchQuery.value.toLowerCase()
    );

    console.log(`Title: ${product.title}, Distance: ${titleDistance}`);
    console.log(`Description: ${product.description}, Distance: ${descriptionDistance}`);

    return titleDistance <= threshold || descriptionDistance <= threshold;
  });
});

function levenshteinDistance(str1, str2) {
  const len1 = str1.length;
  const len2 = str2.length;

  const dp = Array(len1 + 1).fill(null).map(() => Array(len2 + 1).fill(null));

  for (let i = 0; i <= len1; i++) {
    dp[i][0] = i;
  }
  for (let j = 0; j <= len2; j++) {
    dp[0][j] = j;
  }

  for (let i = 1; i <= len1; i++) {
    for (let j = 1; j <= len2; j++) {
      const cost = str1[i - 1] === str2[j - 1] ? 0 : 1;
      dp[i][j] = Math.min(
        dp[i - 1][j] + 1,    // Delete 
        dp[i][j - 1] + 1,    //    Insert
        dp[i - 1][j - 1] + cost //     Substitution
      );
    }
  }

  return dp[len1][len2];
}

const handleSearch = () => {
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