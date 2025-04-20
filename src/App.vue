<script setup>

  import {ref,computed} from 'vue';

  const searchQuery = ref('')
  const cart = ref([])
  const activeTab = ref('menu')
  const paymentMethod = ref('')

  const coffeeList = ref([
    { name: 'Espresso', image: 'https://i.pinimg.com/236x/bc/0c/ff/bc0cffc8b21c24b4b571e98b9ab5da12.jpg', price : 13 },
    { name: 'Latte', image: 'https://i.pinimg.com/474x/f0/65/5f/f0655f2737da76be9b4ac435c65e3d9b.jpg', price : 13}, 
    { name: 'Cappuccino', image: 'https://i.pinimg.com/236x/33/44/2e/33442e58a74503c7cef4fc437a4ebc8e.jpg', price : 14 },
    { name: 'Mocha', image: 'https://i.pinimg.com/236x/1b/2d/77/1b2d77ef227d199b5246c13c40fedbd4.jpg', price : 12 },
    { name: 'Caramel Macchiato', image: 'https://i.pinimg.com/236x/7f/7c/0a/7f7c0a441577459d9a0a01ee28b59cc5.jpg', price : 11 },
    { name: 'Chocolate Hazelnut', image: 'https://i.pinimg.com/474x/6e/d0/a7/6ed0a708dc425ee2208548a5aea27aa9.jpg', price : 15 },
  ]);

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
<div class="app-container">
    <div class="header">
      <h1>Kopeace</h1>
      <div class="nav-tabs">
        <button 
          :class="{ active: activeTab === 'menu' }" 
          @click="activeTab = 'menu'">Menu
        </button>
        <button 
          :class="{ active: activeTab === 'order' }" 
          @click="activeTab = 'order'">
          Order <span v-if="cart.length > 0" class="cart-badge">{{ cart.length }}</span>
        </button>
      </div>
    </div>
  </div>

    <!-- Menu Page -->
    <div v-if="activeTab === 'menu'" class="menu-page">
      <div class="search-bar">
        <input type="text" placeholder="Search for coffee" v-model="searchQuery" />
      </div>

      <div class="coffee-grid">
        <div class="coffee-box" v-for="coffee in filteredCoffeeList" :key="coffee.name">
          <img :src="coffee.image" :alt="coffee.name">
          <div class="coffee-info">
            <p class="coffee-name">{{ coffee.name }}</p>
            <p class="coffee-price">RM{{ coffee.price }}</p>
          </div>
          <button class="add-to-cart" @click="addToCart(coffee)">Add to Cart</button>
        </div>
      </div>
    </div>

  <!-- Order Page -->
  <div v-if="activeTab === 'order'" class="order-page">
      <div class="order-section">
        <h2>Your Order</h2>
        <div v-if="cart.length === 0" class="empty-cart">
          <p>Your cart is empty</p>
          <button @click="activeTab = 'menu'">Browse Menu</button>
        </div>
        <div v-else class="cart-items">
          <div v-for="item in cart" :key="item.name" class="cart-item">
            <img :src="item.image" :alt="item.name" class="cart-item-image">
            <div class="cart-item-details">
              <p class="cart-item-name">{{ item.name }}</p>
              <p class="cart-item-price">RM{{ item.price }}</p>
            </div>
            <div class="quantity-controls">
              <button @click="decreaseQuantity(item)">-</button>
              <span>{{ item.quantity }}</span>
              <button @click="increaseQuantity(item)">+</button>
            </div>
            <p class="item-total">RM{{ item.price * item.quantity }}</p>
            <button class="remove-item" @click="removeFromCart(item)">✕</button>
          </div>
        </div>
      </div>

      <div v-if="cart.length > 0" class="payment-section">
        <div class="payment-methods">
          <h2>Payment Methods</h2>
          <div class="payment-options">
            <label>
              <input type="radio" v-model="paymentMethod" value="cash">
              Cash
            </label>
            <label>
              <input type="radio" v-model="paymentMethod" value="card">
              Credit/Debit Card
            </label>
            <label>
              <input type="radio" v-model="paymentMethod" value="ewallet">
              E-Wallet
            </label>
          </div>
        </div>

        <div class="payment-details">
          <h2>Payment Details</h2>
          <p>Subtotal: RM{{ totalAmount }}</p>
          <p>Tax (6%): RM{{ (totalAmount * 0.06).toFixed(2) }}</p>
          <p class="total-amount">Total Amount: RM{{ (totalAmount * 1.06).toFixed(2) }}</p>
        </div>

        <div class="order-button">
          <h2>RM{{ (totalAmount * 1.06).toFixed(2) }}</h2>
          <button class="place-order" @click="placeOrder">Order Now</button>
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