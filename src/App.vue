<script setup>
import { onMounted, reactive, provide, ref, watch, computed } from 'vue'
import axios from 'axios'
import Header from './components/Header.vue'
import CardList from './components/CardList.vue'

import Drawer from './components/Drawer.vue'
import CartItem from './components/CartItem.vue'
const items = ref([])
const cart = ref([])
const isDrawerOpen = ref(false)

const closeDrawer = () => {
  isDrawerOpen.value = false
}

const addToCart = (item) => {
  cart.value.push(item)
  item.isAdded = true
}

const totalPrice = computed(() => cart.value.reduce((acc, item) => acc + item.price, 0))

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1)
  item.isAdded = false
}

watch(
  cart,
  () => {
    localStorage.setItem('cart', JSON.stringify(cart.value))
  },
  {
    deep: true
  }
)

const createOrder = async () => {
  try {
    const { data } = await axios.post(`https://9f06c5296276a65c.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: totalPrice.value
    })

    cart.value.forEach((element) => {
      element.isAdded = false
    })
    cart.value = []

    return data
  } catch (err) {
    console.log(err)
  }
}

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const openDrawer = () => {
  isDrawerOpen.value = true
}

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

const onChangeSearchInput = (event) => {
  filters.searchQuery = event.target.value
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://9f06c5296276a65c.mokky.dev/favorites')
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.parentId === item.id)

      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (e) {
    console.log(e)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    const { data } = await axios.get('https://604781a0efa572c1.mokky.dev/items', {
      params
    })

    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }))
  } catch (err) {
    console.log(err)
  }
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((CartItem) => CartItem.id === item.id)
  }))
})
watch(filters, fetchItems)

const addToFavorite = async (item) => {
  console.log(item)
  try {
    if (!item.isFavorite) {
      const obj = {
        parentId: item.id
      }

      item.isFavorite = true

      const { data } = await axios.post('https://9f06c5296276a65c.mokky.dev/favorites', obj)
      item.favoriteId = data.id
    } else {
      item.isFavorite = false
      await axios.delete(`https://9f06c5296276a65c.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

provide('cart', {
  cart,
  closeDrawer,
  openDrawer,
  addToCart,
  removeFromCart
})
</script>

<template>
  <Drawer :total-price="totalPrice" v-if="isDrawerOpen" @createOrder="createOrder" />
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Header :total-price="totalPrice" @open-drawer="openDrawer" />
    <div class="p-10">
      <div class="flex justify-between items-center">
        <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>
        <div class="flex gap-4">
          <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
            <option value="name">По названию</option>
            <option value="price">По цене (Дешёвые)</option>
            <option value="-price">По цене (Дорогие)</option>
          </select>
          <div class="relative">
            <img class="absolute left-3 top-3" src="/search.svg" alt="search" />
            <input
              v-on:input="onChangeSearchInput"
              class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
              type="text"
              placeholder="Поиск..."
            />
          </div>
        </div>
      </div>
      <div class="mt-10">
        <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
      </div>
    </div>
  </div>
</template>

<style></style>
