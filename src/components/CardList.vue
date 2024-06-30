<script setup>
import Card from './Card.vue'

defineProps({
  items: Array
})

const onClickAdd = () => {
  console.log('some text')
}
</script>

<template>
  <div class="grid grid-cols-4 gap-5">
    <Card
      v-for="item in items"
      :key="item.id"
      :image-url="item.imageUrl"
      :title="item.title"
      :price="item.price"
      :onClickAdd="onClickAdd"
    />
  </div>
</template>
