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
        <v-list-item
          v-for="(item, index) in sortedTodos"
          :key="item.id"
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
              block
              color="red-accent-4"
              @click="onDeleteButtonClick(item.id)"
            >
              delete
            </v-btn>
          </template>
        </v-list-item>
      </v-list>

      <div class="text-center" v-else>Your haven't any deals yet</div>
    </v-container>
  </main>

</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
const isLoading = ref(false)
const counter = ref(1)
const inputModelValue = ref('')
const todos = ref([])

const sortedTodos = computed(() => {
  return todos.value.sort((a, b) => b.isDone - a.isDone)
})

const loadPage = () => {
  isLoading.value = true
  setTimeout(() => { isLoading.value = false }, 1500)
}

const onAddDealButtonClick = () => {
  todos.value.push({ title: inputModelValue.value, id: counter.value, isDone: false })
  inputModelValue.value = ''
  counter.value += 1
}

const onDeleteButtonClick = (id) => {
  const foundedTodoIndex = todos.value.findIndex(({ id: todoId}) => todoId === id)
  todos.value.splice(foundedTodoIndex, 1)
}

onMounted(loadPage)
</script>
