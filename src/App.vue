<script setup>
import { ref, computed, onMounted } from 'vue';
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

const searchQuery = ref('');
const cart = ref([]);
const activeTab = ref('menu');
const paymentMethod = ref('');

const coffeeList = ref([]);

async function getCoffee() {
  const records = await pb.collection('kopeace').getFullList();

  coffeeList.value = records.map(record => ({
    name: record.Coffee,
    price: record.Price,
    image: record.Picture
  }));
}

onMounted(() => {
  getCoffee();
});

const filteredCoffeeList = computed(() => {
  return coffeeList.value.filter(coffee => {
    return coffee.name.toLowerCase().includes(searchQuery.value.toLowerCase());
  });
});

const totalAmount = computed(() => {
  return cart.value.reduce((total, item) => {
    return total + (item.price * item.quantity);
  }, 0);
});

const addToCart = (coffee) => {
  const existingItem = cart.value.find(item => item.name === coffee.name);
  
  if (existingItem) {
    existingItem.quantity++;
  } else {
    cart.value.push({
      name: coffee.name,
      price: coffee.price,
      image: coffee.image,
      quantity: 1
    });
  }
};

const increaseQuantity = (item) => {
  item.quantity++;
};

const decreaseQuantity = (item) => {
  if (item.quantity > 1) {
    item.quantity--;
  } else {
    removeFromCart(item);
  }
};

const removeFromCart = (item) => {
  const index = cart.value.findIndex(cartItem => cartItem.name === item.name);
  if (index !== -1) {
    cart.value.splice(index, 1);
  }
};

const placeOrder = () => {
  if (cart.value.length === 0) {
    alert('Please add items to your cart before placing an order');
    return;
  }
  
  if (!paymentMethod.value) {
    alert('Please select a payment method');
    return;
  }
  
  alert('Order placed successfully!');
  cart.value = [];
  paymentMethod.value = '';
  activeTab.value = 'menu';
};
</script>

<template>
  <div class="min-h-screen bg-amber-50">
    <!-- Header -->
    <header class="bg-amber-900 text-white shadow-md">
      <div class="container mx-auto px-4 py-4">
        <div class="flex justify-between items-center">
          <h1 class="text-3xl font-bold" style="font-family: 'Playfair Display', serif;">
            Kopeace
            <span class="text-amber-200 text-sm ml-2 italic">Premium Coffee</span>
          </h1>
          
          <div class="flex bg-amber-800 rounded-lg overflow-hidden">
            <button 
              class="px-4 py-2 transition-colors duration-200 flex items-center"
              :class="{ 'bg-amber-700 text-white': activeTab === 'menu', 'text-amber-200 hover:bg-amber-700/50': activeTab !== 'menu' }"
              @click="activeTab = 'menu'"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
              </svg>
              Menu
            </button>
            <button 
              class="px-4 py-2 transition-colors duration-200 flex items-center relative"
              :class="{ 'bg-amber-700 text-white': activeTab === 'order', 'text-amber-200 hover:bg-amber-700/50': activeTab !== 'order' }"
              @click="activeTab = 'order'"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
              </svg>
              Order
              <span 
                v-if="cart.length > 0" 
                class="absolute -top-1 -right-1 bg-amber-200 text-amber-900 text-xs font-bold rounded-full h-5 w-5 flex items-center justify-center"
              >
                {{ cart.length }}
              </span>
            </button>
          </div>
        </div>
      </div>
    </header>

    <!-- Menu Page -->
    <div v-if="activeTab === 'menu'" class="container mx-auto px-4 py-6">
      <div class="mb-6 max-w-md mx-auto">
        <div class="relative">
          <input 
            type="text" 
            placeholder="Search for coffee..." 
            v-model="searchQuery"
            class="w-full px-4 py-3 rounded-full border-2 border-amber-200 focus:border-amber-400 focus:outline-none focus:ring-2 focus:ring-amber-100 bg-white pl-10"
          />
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 absolute left-3 top-3.5 text-amber-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
          </svg>
        </div>
      </div>

      <div v-if="filteredCoffeeList.length === 0" class="text-center py-12">
        <p class="text-lg text-amber-800">Loading coffee options or no matches found...</p>
      </div>

      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
        <div 
          v-for="coffee in filteredCoffeeList" 
          :key="coffee.name"
          class="bg-white rounded-xl shadow-md overflow-hidden transform transition-all duration-300 hover:shadow-lg"
        >
          <div class="relative h-48">
            <img 
              :src="coffee.image" 
              :alt="coffee.name"
              class="w-full h-full object-cover"
            />
            <div class="absolute inset-0 bg-gradient-to-t from-amber-900/60 to-transparent opacity-0 hover:opacity-100 transition-opacity duration-300 flex items-end">
              <div class="p-4 w-full">
                <button 
                  @click="addToCart(coffee)"
                  class="w-full bg-amber-200 hover:bg-amber-300 text-amber-900 font-medium rounded-lg py-2 transition-colors duration-200 flex items-center justify-center"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
                  </svg>
                  Add to Cart
                </button>
              </div>
            </div>
          </div>
          
          <div class="p-4">
            <div class="flex justify-between items-center">
              <h3 class="text-xl font-semibold text-amber-900">{{ coffee.name }}</h3>
              <p class="text-amber-700 font-medium">RM{{ coffee.price }}</p>
            </div>
            <button 
              @click="addToCart(coffee)"
              class="mt-3 w-full bg-amber-100 hover:bg-amber-200 text-amber-800 font-medium rounded-lg py-2 transition-colors duration-200 flex items-center justify-center shadow-sm"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
              </svg>
              Add to Cart
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Order Page -->
    <div v-if="activeTab === 'order'" class="container mx-auto px-4 py-6">
      <div class="grid grid-cols-1 lg:grid-cols-5 gap-6">
        <!-- Cart Section - 3 columns -->
        <div class="lg:col-span-3 bg-white rounded-xl shadow-md p-6">
          <h2 class="text-2xl font-semibold text-amber-900 border-b border-amber-100 pb-4 mb-4">Your Order</h2>
          
          <div v-if="cart.length === 0" class="text-center py-12">
            <p class="text-lg text-amber-700 mb-4">Your cart is empty</p>
            <button 
              @click="activeTab = 'menu'"
              class="bg-amber-600 hover:bg-amber-700 text-white px-6 py-2 rounded-lg font-medium transition-colors duration-200"
            >
              Browse Menu
            </button>
          </div>
          
          <div v-else class="space-y-4">
            <div 
              v-for="item in cart" 
              :key="item.name" 
              class="flex items-center bg-amber-50 rounded-lg overflow-hidden shadow-sm"
            >
              <img 
                :src="item.image" 
                :alt="item.name" 
                class="h-20 w-20 object-cover"
              >
              
              <div class="flex-grow p-3">
                <p class="font-medium text-amber-900">{{ item.name }}</p>
                <p class="text-sm text-amber-700">RM{{ item.price }}</p>
              </div>
              
              <div class="flex items-center px-3">
                <div class="flex items-center bg-white rounded-lg border border-amber-200">
                  <button 
                    @click="decreaseQuantity(item)"
                    class="px-3 py-1 text-amber-700 hover:bg-amber-100 transition-colors duration-200"
                  >
                    -
                  </button>
                  <span class="px-3 py-1 font-medium text-amber-900">{{ item.quantity }}</span>
                  <button 
                    @click="increaseQuantity(item)"
                    class="px-3 py-1 text-amber-700 hover:bg-amber-100 transition-colors duration-200"
                  >
                    +
                  </button>
                </div>
              </div>
              
              <div class="px-4 font-medium text-amber-900">
                RM{{ (item.price * item.quantity).toFixed(2) }}
              </div>
              
              <button 
                @click="removeFromCart(item)"
                class="p-3 text-amber-500 hover:text-amber-700 transition-colors duration-200"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>
        </div>
        
        <!-- Payment Section - 2 columns -->
        <div v-if="cart.length > 0" class="lg:col-span-2 space-y-6">
          <!-- Payment Methods -->
          <div class="bg-white rounded-xl shadow-md p-6">
            <h2 class="text-xl font-semibold text-amber-900 border-b border-amber-100 pb-3 mb-4">Payment Method</h2>
            
            <div class="space-y-3">
              <label class="flex items-center p-3 border border-amber-200 rounded-lg hover:bg-amber-50 cursor-pointer transition-colors duration-200">
                <input 
                  type="radio" 
                  v-model="paymentMethod" 
                  value="cash"
                  class="mr-3 text-amber-600 focus:ring-amber-500"
                >
                <span class="text-amber-900">Cash</span>
              </label>
              
              <label class="flex items-center p-3 border border-amber-200 rounded-lg hover:bg-amber-50 cursor-pointer transition-colors duration-200">
                <input 
                  type="radio" 
                  v-model="paymentMethod" 
                  value="card"
                  class="mr-3 text-amber-600 focus:ring-amber-500"
                >
                <span class="text-amber-900">Credit/Debit Card</span>
              </label>
              
              <label class="flex items-center p-3 border border-amber-200 rounded-lg hover:bg-amber-50 cursor-pointer transition-colors duration-200">
                <input 
                  type="radio" 
                  v-model="paymentMethod" 
                  value="ewallet"
                  class="mr-3 text-amber-600 focus:ring-amber-500"
                >
                <span class="text-amber-900">E-Wallet</span>
              </label>
            </div>
          </div>
          
          <!-- Order Summary -->
          <div class="bg-white rounded-xl shadow-md p-6">
            <h2 class="text-xl font-semibold text-amber-900 border-b border-amber-100 pb-3 mb-4">Order Summary</h2>
            
            <div class="space-y-2 mb-6">
              <div class="flex justify-between text-amber-800">
                <span>Subtotal</span>
                <span>RM{{ totalAmount.toFixed(2) }}</span>
              </div>
              <div class="flex justify-between text-amber-800">
                <span>Tax (6%)</span>
                <span>RM{{ (totalAmount * 0.06).toFixed(2) }}</span>
              </div>
              <div class="border-t border-amber-200 pt-2 mt-2 flex justify-between font-medium text-amber-900">
                <span>Total</span>
                <span class="text-lg">RM{{ (totalAmount * 1.06).toFixed(2) }}</span>
              </div>
            </div>
            
            <button 
              @click="placeOrder"
              class="w-full bg-amber-600 hover:bg-amber-700 text-white font-medium py-3 rounded-lg shadow-sm transition-colors duration-200 flex items-center justify-center"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
              </svg>
              Place Order - RM{{ (totalAmount * 1.06).toFixed(2) }}
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Footer -->
    <footer class="mt-12 py-6 bg-amber-900 text-amber-200">
      <div class="container mx-auto px-4 text-center">
        <p>© 2025 Kopeace Coffee Shop | Crafted with love for coffee enthusiasts</p>
      </div>
    </footer>
  </div>
</template>