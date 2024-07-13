<script setup>
const emit = defineEmits(['onClickRemove'])
defineProps({
  id: Number,
  title: String,
  imageUrl: String,
  price: Number
})
</script>
<template>
  <div class="flex border items-center border-slate-100 p-4 rounded-xl gap-4">
    <img class="w1-16 h-16" :src="imageUrl" :alt="title" />
    <div class="flex flex-col justify-between">
      <p>{{ title }}</p>
      <div class="flex justify-between mt-2 flex-1">
        <b class="flex-1">{{ price }} руб.</b>
        <img
          class="opacity-40 hover:opacity-100 cursor-pointer transition"
          src="/close.svg"
          alt=""
          @click="emit('onClickRemove')"
        />
      </div>
    </div>
  </div>
</template>
