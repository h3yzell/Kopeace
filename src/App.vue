<script setup>

  import {ref,computed} from 'vue';

  const searchQuery = ref('')

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

</script>

<template>

  <div>
  <h1>Kopeace</h1>

  <input type="text" placeholder="Search for coffee" v-model="searchQuery" />
  </div>

  <div class="coffee-grid">
    <div class="coffee-box" v-for="coffee in filteredCoffeeList" :key="coffee.name">
      <img :src="coffee.image" :alt="coffee.name">
        <p class="coffee-name">{{ coffee.name }}</p>
        <p class ="coffee-price" > RM{{ coffee.price }}</p>
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