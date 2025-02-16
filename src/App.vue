<template>
  <main>
    <div v-if="isLoading" class="text-center">Loading...</div>

    <v-container v-else class="bg-surface-variant">
      <v-row no-gutters>
       <v-col>
         <v-form @submit.prevent>
           <v-container>
             <v-row class="align-center">
               <v-col>
                 <v-text-field
                   v-model="inputModelValue"
                   label="enter your deal"
                 ></v-text-field>
               </v-col>

               <v-col cols="2">
                 <v-btn
                   type="submit"
                   block
                   @click="onAddDealButtonClick"
                 >
                   Add deal
                 </v-btn>
               </v-col>
             </v-row>
           </v-container>
         </v-form>
       </v-col>
      </v-row>

      <v-list
        v-if="todos.length"
        item-value="id"
        bg-color="transparent"
      >
        <div
          v-for="(item, index) in todos"
          :key="item.id"
          @drop="onDrop($event, item.id)"
          @dragenter="onDragEnter(item.id)"
          @dragover.prevent
        >

          <v-divider v-if="index !== 0 && item.id !== droppedTodoId" />
          <v-divider
            v-else-if="transferringTodoId !== item.id && (index !== 0 && item.id === droppedTodoId) || (index === 0 && item.id === droppedTodoId)"
            :thickness="5"
            class="border-opacity-50 rounded"
            color="success"
          />

          <v-list-item
            :ref="(el) => dragImages[item.id] = el"
            :value="item"
          >
            <template #prepend>
             <div class="d-flex ga-3 align-center">
               #{{ index + 1 }}

               <v-list-item-action class="pt-6" start>
                 <v-checkbox
                   v-model="item.isDone"
                   color="info"
                 />
               </v-list-item-action>
             </div>
            </template>

            {{ item.title }}

            <template #append>
              <v-btn
                icon="mdi-delete"
                color="red-accent-4"
                title="delete"
                aria-label="delete todo item"
                @click="onDeleteButtonClick(item.id)"
              />
              <v-btn
                icon="mdi-drag"
                variant="plain"
                size="x-large"
                title="transfer"
                aria-label="transfer todo item"
                draggable="true"
                @dragstart="onDragStart($event, item.id)"
                @dragend="onDragEnd"
              />
            </template>
          </v-list-item>
        </div>
      </v-list>

      <div class="text-center" v-else>Your haven't any deals yet</div>
    </v-container>
  </main>

</template>

<script setup>
import { ref, onMounted } from 'vue'
const isLoading = ref(false)
const counter = ref(1)
const inputModelValue = ref('')
const todos = ref([])
const dragImages = ref([])
const transferringTodoId = ref(null)
const droppedTodoId = ref(null)

const loadPage = () => {
  isLoading.value = true
  setTimeout(() => { isLoading.value = false }, 1500)
}

const onAddDealButtonClick = () => {
  todos.value.push({ title: inputModelValue.value, id: String(counter.value), isDone: false })
  inputModelValue.value = ''
  counter.value += 1
}

const onDeleteButtonClick = (id) => {
  const foundedTodoIndex = todos.value.findIndex(({ id: todoId}) => todoId === id)
  todos.value.splice(foundedTodoIndex, 1)
}

const onDragStart = (evt, id) => {
  const dragImage = dragImages.value[id].$el
  const buttonHeight = evt.target.clientHeight
  const buttonWidth = evt.target.clientWidth
  const topGapToButton = (dragImage.clientHeight - buttonHeight) / 2
  const rightGapToButton = 16
  transferringTodoId.value = id

  evt.dataTransfer.dropEffect = 'move'
  evt.dataTransfer.effectAllowed = 'move'
  evt.dataTransfer.setDragImage(dragImage, dragImage.clientWidth - (buttonWidth - evt.offsetX) - rightGapToButton, topGapToButton + evt.offsetY);
  evt.dataTransfer.setData('itemId', id)
}

const onDrop = (evt, droppedId) => {
  const transferredId = evt.dataTransfer.getData('itemId')
  const transferredTodoItem = todos.value.find(({ id }) => id === transferredId)

  todos.value = todos.value.reduce((acc, item, index) => {
    const isLast = todos.value.length === index + 1
    const currentTodoId = item.id

    // по дефолту вставляем перетаскиваемый элемент выше дропнутого.
    // Проблема: Получаем ситуацию, что невозможно перенести перетаскиваемый элемент на последнюю позицию.
    // Решение: Флаг isLast позволяет перенести транспортированный элемент на последнюю позицию в списке.
    if (!isLast && currentTodoId === droppedId) acc.push(transferredTodoItem)
    if (currentTodoId !== transferredId) acc.push(item)
    if (isLast && currentTodoId === droppedId) acc.push(transferredTodoItem)

    return acc
  }, [])
}

const onDragEnter = (id) => {
  droppedTodoId.value = id
}

const onDragEnd = () => {
  dragImages.value = []
  transferringTodoId.value = null
  droppedTodoId.value = null
}

onMounted(loadPage)
</script>
