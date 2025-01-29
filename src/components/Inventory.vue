<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import { ref, computed, watch, onMounted, reactive } from 'vue'

const slots = ref([
  {
    item: reactive({
      image: '/public/item-1.png',
      count: 4,
      name: 'Предмет 1',
      description: `Lorem ipsum dolor sit amet consectetur adipisicing elit. Eos sapiente commodi aspernatur possimus suscipit doloremque tempora quos explicabo officia! Voluptas optio!`,
    }),
  },
  {
    item: reactive({
      image: '/public/item-2.png',
      count: 2,
      name: 'Предмет 2',
      description: `Lorem ipsum dolor sit amet consectetur adipisicing elit. Eos sapiente commodi aspernatur possimus suscipit doloremque tempora quos explicabo officia! Voluptas optio!`,
    }),
  },
  {
    item: reactive({
      image: '/public/item-3.png',
      count: 5,
      name: 'Предмет 3',
      description: `Lorem ipsum dolor sit amet consectetur adipisicing elit. Eos sapiente commodi aspernatur possimus suscipit doloremque tempora quos explicabo officia! Voluptas optio!`,
    }),
  },
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
const selectedItem = ref(null)
const isInfoOpen = ref(false)
const isDeleteConfirmOpen = ref(false)
const deleteQuantity = ref('')
const gridRef = ref(null)
const gridRect = ref({ top: 0, height: 0 })

const saveToLocalStorage = () => {
  localStorage.setItem('inventorySlots', JSON.stringify(slots.value))
}

const loadFromLocalStorage = () => {
  const savedSlots = localStorage.getItem('inventorySlots')
  if (savedSlots) {
    slots.value = JSON.parse(savedSlots)
  }
}

watch(slots, saveToLocalStorage, { deep: true })

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
  }),
)

const onDragStart = (event, index) => {
  if (!slots.value[index].item) {
    event.preventDefault()
    return
  }

  event.dataTransfer.setData('draggedIndex', index.toString())
  event.target.classList.add('dragging')
}

const onDragEnd = (event) => {
  event.target.classList.remove('dragging')
}

const onDrop = (event, targetIndex) => {
  event.preventDefault()

  const draggedIndex = parseInt(event.dataTransfer.getData('draggedIndex'), 10)
  if (!isNaN(draggedIndex) && draggedIndex !== targetIndex) {
    if (slots.value[targetIndex].item === null) {
      slots.value[targetIndex].item = slots.value[draggedIndex].item
      slots.value[draggedIndex].item = null
    } else {
      const temp = slots.value[targetIndex].item
      slots.value[targetIndex].item = slots.value[draggedIndex].item
      slots.value[draggedIndex].item = temp
    }
  }
}

const onDragOver = (event) => {
  event.preventDefault()
}

const openItemInfo = (item, index) => {
  if (item) {
    selectedItem.value = { ...item, index }
    console.log('Selected Item:', JSON.parse(JSON.stringify(selectedItem.value)))
    isInfoOpen.value = true
  }
}

const closeItemInfo = () => {
  isInfoOpen.value = false
  setTimeout(() => {
    selectedItem.value = null
    isDeleteConfirmOpen.value = false
    deleteQuantity.value = ''
  }, 300)
}

const openDeleteConfirm = () => {
  isDeleteConfirmOpen.value = true
}

const closeDeleteConfirm = () => {
  isDeleteConfirmOpen.value = false
  deleteQuantity.value = ''
}

const confirmDelete = () => {
  const quantity = parseInt(deleteQuantity.value)
  if (quantity > 0 && selectedItem.value && quantity <= selectedItem.value.count) {
    const index = selectedItem.value.index
    slots.value[index].item.count -= quantity
    if (slots.value[index].item.count === 0) {
      slots.value[index].item = null
    }
    closeItemInfo()
  }
}

const updateGridRect = () => {
  if (gridRef.value) {
    const rect = gridRef.value.getBoundingClientRect()
    const parentRect = gridRef.value.offsetParent.getBoundingClientRect()
    gridRect.value = {
      top: rect.top - parentRect.top,
      right: parentRect.right - rect.right,
      height: rect.height,
    }
  }
}

onMounted(() => {
  loadFromLocalStorage()
  updateGridRect()
  window.addEventListener('resize', updateGridRect)
})
</script>

<template>
  <div class="relative flex items-center justify-center min-h-screen w-full">
    <div class="bg-[#1D1D1D] max-w-[850px] w-full p-8 flex flex-col">
      <div class="flex items-center justify-center gap-6 relative">
        <div class="bg-[#262626] border border-[#4D4D4D] rounded-lg py-[18px] px-4">
          <img class="rounded-md object-cover" src="/public/img-blur.png" alt="#" width="208px" height="240px"/>
          <h2 class="text-xl text-center mt-1">Инвентарь</h2>
          <p class="text-center max-w-[200px] mt-1 overflow-y-auto max-h-[192px]">
            Lorem ipsum dolor sit amet consectetur adipisicing elit. Eos sapiente commodi aspernatur
            possimus suscipit doloremque tempora quos explicabo officia! Voluptas optio!
          </p>
        </div>
        <div class="grid grid-cols-5 w-fit rounded-lg" ref="gridRef">
          <div
            v-for="(slot, index) in slots"
            :key="index"
            :class="`w-[100px] h-[100px] border border-[#4D4D4D] flex items-center justify-center bg-[#262626] cursor-pointer hover:bg-[#2F2F2F] ${slotClasses[index]}`"
            @dragover="onDragOver"
            @drop="onDrop($event, index)"
            @click="openItemInfo(slot.item, index)"
          >
            <div
              v-if="slot.item"
              class="w-full h-full flex items-center justify-center relative"
              draggable="true"
              @dragstart="onDragStart($event, index)"
              @dragend="onDragEnd"
            >
              <img :src="slot.item.image" alt="Item" class="w-12 h-12 object-contain" />
              <span
                class="absolute bottom-0 right-0 text-[9px] text-white opacity-40 px-1.5 py-0.5 rounded-sm border border-[#4D4D4D] rounded-tl-md"
              >
                {{ slot.item.count }}
              </span>
            </div>
          </div>
        </div>

        <Transition name="slide">
          <div
            v-if="selectedItem && isInfoOpen"
            class="absolute right-0 w-60 bg-[#262626] border border-l border-[#4D4D4D] overflow-y-auto rounded-r-lg flex flex-col"
            :style="{ top: '2.5px', height: '99%', transform: 'translateX(-10px)' }"
          >
            <div class="p-6 overflow-y-hidden">
              <div class="flex justify-between items-start mb-4">
                <h3 class="text-xl font-bold"></h3>
                <button
                  @click="closeItemInfo"
                  class="text-gray-400 hover:text-white cursor-pointer"
                >
                  <svg
                    width="12"
                    height="12"
                    viewBox="0 0 12 12"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                  >
                    <path
                      d="M12 1.05L10.95 0L6 4.95L1.05 0L0 1.05L4.95 6L0 10.95L1.05 12L6 7.05L10.95 12L12 10.95L7.05 6L12 1.05Z"
                      fill="white"
                    />
                  </svg>
                </button>
              </div>
              <div class="flex flex-col">
                <img
                  :src="selectedItem.image"
                  alt="Item"
                  class="w-24 h-24 object-contain mx-auto mb-4"
                />
                <div class="border-y-1 py-2 border-[#4D4D4D] flex-1">
                  <h3 class="text-2xl font-bold text-center">
                    {{ selectedItem.name }}
                  </h3>
                  <p
                    :class="{
                      'max-h-[100px]': isDeleteConfirmOpen,
                      'max-h-[200px]': !isDeleteConfirmOpen,
                    }"
                    class="text-sm mb-4 mt-2 text-center overflow-y-auto"
                  >
                    {{ selectedItem.description }}
                  </p>
                  <p class="text-sm mb-4 text-center">Количество: {{ selectedItem.count }}</p>
                </div>
                <button
                  v-if="!isDeleteConfirmOpen"
                  @click="openDeleteConfirm"
                  class="mt-4 w-full bg-[#FA7272] text-sm py-2 rounded-md hover:bg-[#FF8888] transition-colors cursor-pointer"
                >
                  Удалить предмет
                <!-- TODO: добавить вывод ошибки если пользователь ввел некорректное количество -->
                  
                </button>
                <div v-else class="space-y-2">
                  <input
                    v-model="deleteQuantity"
                    type="number"
                    placeholder="Введите количество"
                    class="w-full bg-[#1D1D1D] border border-[#4D4D4D] rounded px-3 py-2 text-sm"
                    min="1"
                    :max="selectedItem.count"
                  />
                  <div class="flex space-x-3">
                    <button
                      @click="closeDeleteConfirm"
                      class="mt-4 w-full bg-white text-sm text-[#2D2D2D] py-2 px-2 rounded-md hover:bg-[#CFCFCF] transition-colors cursor-pointer"
                    >
                      Отмена
                    </button>
                    <button
                      @click="confirmDelete"
                      class="mt-4 w-full bg-[#FA7272] text-sm py-2 px-2 rounded-md hover:bg-[#FF8888] transition-colors cursor-pointer"
                    >
                      Подтвердить
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </Transition>
      </div>
      <div class="relative bg-[#262626] border border-[#4D4D4D] rounded-lg py-4 px-4 mt-6 flex flex-col gap-2">
        <input class="max-w-11/12 w-full gradient rounded-md py-1 px-4" type="text" />
        <div class="absolute top-2.5 right-3 cursor-pointer">
          <svg
            width="12"
            height="12"
            viewBox="0 0 12 12"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M12 1.05L10.95 0L6 4.95L1.05 0L0 1.05L4.95 6L0 10.95L1.05 12L6 7.05L10.95 12L12 10.95L7.05 6L12 1.05Z"
              fill="white"
            />
          </svg>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.grid > div {
  box-sizing: border-box;
}
.dragging {
  opacity: 0.5;
}
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}
.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}
.slide-enter-to,
.slide-leave-from {
  transform: translateX(0);
}

.gradient {
  background: linear-gradient(90deg, #3c3c3c 2%, #444444 51%, #333333 99%);
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideOut {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(100%);
    opacity: 0;
  }
}

.slide-enter-active {
  animation: slideIn 0.4s ease forwards;
}

.slide-leave-active {
  animation: slideOut 0.4s ease forwards;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
}

p::-webkit-scrollbar {
  width: 8px;
}

p::-webkit-scrollbar-track {
  background: #333;
  border-radius: 10px;
}

p::-webkit-scrollbar-thumb {
  background-color: #888;
  border-radius: 10px;
  border: 2px solid #333;
}

p::-webkit-scrollbar-thumb:hover {
  background-color: #555;
}
</style>
