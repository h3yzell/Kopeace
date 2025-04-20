<script setup>

  import {ref,computed, onMounted} from 'vue';
  import PocketBase from 'pocketbase';

  const pb = new PocketBase('http://127.0.0.1:8090');

  const searchQuery = ref('')
  const cart = ref([])
  const activeTab = ref('menu')
  const paymentMethod = ref('')

  const coffeeList = ref([]);

  async function getCoffee(){
    const records = await pb.collection('kopeace').getFullList();

    coffeeList.value = records.map(record => ({
      name: record.Coffee,
      price: record.price,
      image: record.picture
    }))
  }

  onMounted(() => {
    getCoffee()
  })

  const filteredCoffeeList = computed(() => {
    return coffeeList.value.filter(coffee => {
      return coffee.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    })
  })

  const totalAmount = computed(() => {
    return cart.value.reduce((total, item) => {
      return total + (item.price * item.quantity)
    }, 0)
  })

  const addToCart = (coffee) => {
  const existingItem = cart.value.find(item => item.name === coffee.name)
  
  if (existingItem) {
    existingItem.quantity++
  } else {
    cart.value.push({
      name: coffee.name,
      price: coffee.price,
      image: coffee.image,
      quantity: 1
    })
  }
}

const increaseQuantity = (item) => {
  item.quantity++
}

const decreaseQuantity = (item) => {
  if (item.quantity > 1) {
    item.quantity--
  } else {
    removeFromCart(item)
  }
}

const removeFromCart = (item) => {
  const index = cart.value.findIndex(cartItem => cartItem.name === item.name)
  if (index !== -1) {
    cart.value.splice(index, 1)
  }
}

const placeOrder = () => {
  if (cart.value.length === 0) {
    alert('Please add items to your cart before placing an order')
    return
  }
  
  if (!paymentMethod.value) {
    alert('Please select a payment method')
    return
  }
  
  alert('Order placed successfully!')
  cart.value = []
  paymentMethod.value = ''
  activeTab.value = 'menu'
}
</script>

<template>
  <div class="min-h-screen bg-[#f7f2ee] font-sans text-[#3e2723]">
    <link rel="icon" type="image/png" href="/kopeace.png" />

<!-- Header -->
<div class="header flex items-center justify-between p-4 bg-white shadow-md">
  <div class="flex items-center space-x-3">
    <img src="/kopeace.png" alt="Kopeace Logo" class="h-10 w-10 object-cover rounded-full" />
    <h1 class="text-2xl font-bold text-brown-800">Kopeace</h1>
  </div>

  <div class="nav-tabs flex space-x-2">
    <button 
      :class="{ 'font-semibold border-b-2 border-brown-600': activeTab === 'menu' }" 
      class="px-4 py-2 text-brown-600 hover:text-brown-800" 
      @click="activeTab = 'menu'">
      Menu
    </button>
    <button 
      :class="{ 'font-semibold border-b-2 border-brown-600': activeTab === 'order' }" 
      class="relative px-4 py-2 text-brown-600 hover:text-brown-800" 
      @click="activeTab = 'order'">
      Order 
      <span v-if="cart.length > 0" class="absolute -top-2 -right-2 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">
        {{ cart.length }}
      </span>
    </button>
  </div>
</div>




    <!-- Menu Page -->
    <div v-if="activeTab === 'menu'" class="p-6">
      <div class="max-w-md mx-auto mb-6">
        <input 
          type="text" 
          placeholder="Search your brew..." 
          v-model="searchQuery"
          class="w-full px-4 py-2 rounded-full border border-[#bcaaa4] bg-white placeholder-[#8d6e63] focus:ring-2 focus:ring-[#6d4c41]"
        />
      </div>

      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        <div 
          v-for="coffee in filteredCoffeeList" 
          :key="coffee.name"
          class="bg-white rounded-xl shadow-md overflow-hidden transition hover:scale-105"
        >
          <img :src="coffee.image" :alt="coffee.name" class="h-48 w-full object-cover">
          <div class="p-4 text-center">
            <h2 class="text-xl font-semibold">{{ coffee.name }}</h2>
            <p class="text-[#795548] text-lg">RM{{ coffee.price }}</p>
            <button 
              class="mt-4 px-4 py-2 bg-[#6d4c41] text-white rounded-full hover:bg-[#5d4037] transition"
              @click="addToCart(coffee)"
            >
              Add to Cart
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Order Page -->
    <div v-if="activeTab === 'order'" class="p-6">
      <div class="max-w-4xl mx-auto">
        <h2 class="text-2xl font-bold mb-4">🧾 Your Order</h2>

        <div v-if="cart.length === 0" class="text-center py-10 bg-white rounded-lg shadow">
          <p class="mb-4 text-lg">Your cart is empty</p>
          <button 
            class="px-4 py-2 bg-[#6d4c41] text-white rounded-full hover:bg-[#5d4037]"
            @click="activeTab = 'menu'">
            Browse Menu
          </button>
        </div>

        <div v-else class="space-y-4">
          <div 
            v-for="item in cart" 
            :key="item.name"
            class="flex items-center justify-between bg-white p-4 rounded-lg shadow"
          >
            <div class="flex items-center gap-4">
              <img :src="item.image" class="w-16 h-16 object-cover rounded-md" />
              <div>
                <p class="font-semibold">{{ item.name }}</p>
                <p class="text-[#8d6e63]">RM{{ item.price }}</p>
              </div>
            </div>

            <div class="flex items-center gap-2">
              <button @click="decreaseQuantity(item)" class="bg-[#d7ccc8] w-8 h-8 rounded-full">-</button>
              <span>{{ item.quantity }}</span>
              <button @click="increaseQuantity(item)" class="bg-[#d7ccc8] w-8 h-8 rounded-full">+</button>
            </div>

            <p class="font-semibold">RM{{ item.price * item.quantity }}</p>
            <button @click="removeFromCart(item)" class="text-red-500 text-lg">✕</button>
          </div>

          <!-- Payment Section -->
          <div class="bg-white p-6 rounded-lg shadow space-y-6 mt-6">
            <div>
              <h2 class="text-xl font-bold">💳 Payment Method</h2>
              <div class="space-x-4 mt-2">
                <label class="inline-flex items-center gap-2">
                  <input type="radio" v-model="paymentMethod" value="cash"> Cash
                </label>
                <label class="inline-flex items-center gap-2">
                  <input type="radio" v-model="paymentMethod" value="card"> Card
                </label>
                <label class="inline-flex items-center gap-2">
                  <input type="radio" v-model="paymentMethod" value="ewallet"> E-Wallet
                </label>
              </div>
            </div>

            <div>
              <h2 class="text-xl font-bold">📋 Payment Details</h2>
              <p>Subtotal: RM{{ totalAmount }}</p>
              <p>Tax (6%): RM{{ (totalAmount * 0.06).toFixed(2) }}</p>
              <p class="font-bold text-lg">Total: RM{{ (totalAmount * 1.06).toFixed(2) }}</p>
            </div>

            <div class="text-right">
              <button 
                class="px-6 py-3 bg-[#6d4c41] text-white rounded-full text-lg hover:bg-[#5d4037]"
                @click="placeOrder">
                Order Now - RM{{ (totalAmount * 1.06).toFixed(2) }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<style scoped>
.coffee-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  padding: 20px;
  width: 100%;
  box-sizing: border-box;
}

.coffee-box {
  background-color: #fff;
  border-radius: 12px;
  padding: 15px;
  text-align: center;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.coffee-box:hover {
  transform: scale(1.05);
}

.coffee-box img {
  width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 8px;
}

.coffee-name {
  margin-top: 10px;
  font-size: 18px;
  font-weight: bold;
}

.coffee-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  padding: 20px;
}

.coffee-box {
  background-color: #fff;
  border-radius: 12px;
  padding: 15px;
  text-align: center;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.coffee-box img {
  width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 8px;
}

.coffee-name {
  margin-top: 10px;
  font-size: 18px;
  font-weight: bold;
}
</style>