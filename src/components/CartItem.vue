<template>
  <div class="flex items-center border border-slate-100 p-4 rounded-xl gap-4">
    <img
      class="w-16 h-16 object-contain"
      :src="imageUrl"
      :alt="title"
    />

    <div class="flex flex-col flex-1">
      <p>{{ title }}</p>

      <div class="flex justify-between mt-2">
        <b class="flex-1">{{ price }} руб.</b>
        <img
          class="opacity-30 hover:opacity-100 cursor-pointer transition"
          src="/close.svg"
          alt="Close"
          @click="emit('onClickRemove')"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
  defineProps({
    id: Number,
    title: String,
    imageUrl: String,
    price: Number,
  });

  const emit = defineEmits(['onClickRemove']);
</script>
