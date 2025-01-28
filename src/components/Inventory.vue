<script setup>
import { ref, computed } from 'vue'

const slots = ref([
  { item: { image: '/public/item-1.png', count: 4 } },
  { item: { image: '/public/item-2.png', count: 2 } },
  { item: { image: '/public/item-3.png', count: 5 } },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
  { item: null },
])

const columns = 5

const slotClasses = computed(() =>
  slots.value.map((_, index) => {
    const isFirstRow = index < columns
    const isLastRow = index >= slots.value.length - columns
    const isFirstColumn = index % columns === 0
    const isLastColumn = (index + 1) % columns === 0

    return [
      isFirstRow && isFirstColumn ? 'rounded-tl-lg' : '',
      isFirstRow && isLastColumn ? 'rounded-tr-lg' : '',
      isLastRow && isFirstColumn ? 'rounded-bl-lg' : '',
      isLastRow && isLastColumn ? 'rounded-br-lg' : '',
    ]
      .filter(Boolean)
      .join(' ')
  })
)
</script>

<template>
  <div class="flex items-center justify-center min-h-screen w-full">
    <div class="bg-[#1D1D1D] max-w-[850px] w-full p-8 flex items-center gap-6">
      <div class="bg-[#262626] border border-[#4D4D4D] rounded-lg py-[18px] px-4">
        <img class="rounded-md" src="/public/img-blur.png" alt="#" />
        <h2 class="text-xl text-center mt-1">Inventory</h2>
        <p class="text-center max-w-[200px] mt-1">
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Eos sapiente commodi aspernatur
          possimus suscipit doloremque tempora quos explicabo officia! Voluptas optio!
        </p>
      </div>

      <div class="grid grid-cols-5 p-4 w-fit rounded-lg">
        <div
          v-for="(slot, index) in slots"
          :key="index"
          :class="`w-[100px] h-[100px] border border-[#4D4D4D] flex items-center justify-center bg-[#262626] cursor-pointer hover:bg-[#2F2F2F] ${slotClasses[index]}`"
        >
          <div v-if="slot.item" class="w-full h-full flex items-center justify-center relative">
            <img :src="slot.item.image" alt="Item" class="w-12 h-12 object-contain" />
            <span
              class="absolute bottom-0 right-0 text-[9px] text-white opacity-40 px-1.5 py-0.5 rounded-sm border border-[#4D4D4D] rounded-tl-md"
            >
              {{ slot.item.count }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.grid > div {
  box-sizing: border-box;
}
</style>