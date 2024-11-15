<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue';

const nameDefault = 'Aisha';

type ITodo = {
  content: string;
  category: string;
  done: boolean;
  createdAt: number;
};

const todos = ref<ITodo[]>([]);
const name = ref<string>('');

const input_content = ref<string>('');
const input_category = ref<string>('');


const todos_asc = computed(() => [...todos.value].sort(
  (a, b) => Number(b.createdAt) - Number(a.createdAt)
));

const addTodo = () => {
  if (input_content.value.trim() === '') {
    alert('Напишите задачу!');
    return;
  }

  if (input_category.value.trim() === '') {
    alert('Выберите категорию!');
    return;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),
  });

  input_content.value = '';
  input_category.value = '';
};

const removeTodo = (todoDeleted: ITodo) => {
  todos.value = todos.value.filter((todo) => todo !== todoDeleted);
};


watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal));
}, { deep: true });


watch(name, (newVal) => {
  localStorage.setItem('name', newVal.trim() !== '' ? newVal : nameDefault);
});


onMounted(() => {
  name.value = localStorage.getItem('name') || nameDefault;
  try {
    const todosFromStorage = localStorage.getItem('todos');
    todos.value = todosFromStorage ? JSON.parse(todosFromStorage) as ITodo[] : [];
  } catch (error) {
    console.error('Ошибка при чтении todos из localStorage:', error);
    todos.value = [];
  }
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        Привет,
        <input type="text" placeholder="Введите своё имя здесь" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>Управление задачами</h3>

      <form @submit.prevent="addTodo">
        <h4>Что у вас в списке задач!?</h4>
        <input type="text" placeholder="например: создать видео" v-model="input_content" />

        <h4>Выберите категорию.</h4>
        <div class="options">
          <label>
            <input type="radio" name="category" value="business" v-model="input_category" />
            <span class="bubble business"></span>
            <div>Работа</div>
          </label>

          <label>
            <input type="radio" name="category" value="personal" v-model="input_category" />
            <span class="bubble personal"></span>
            <div>Личное</div>
          </label>
        </div>

        <input type="submit" value="Добавить задачу">
      </form>
    </section>

    <section class="todo-list">
      <h3>Список задач</h3>
      <div class="list">
        <div v-for="todo in todos_asc" :key="todo.createdAt" :class="`todo-item ${todo.done && 'done'}`">
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Удалить</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
