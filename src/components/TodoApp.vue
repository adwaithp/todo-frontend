<template>
  <div class="bg-gray-100 p-4">
    <div class="max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
      <h1 class="text-3xl font-semibold mb-4">Todo App</h1>
      <div class="flex mb-4">
        <input
          v-model="newTodo.title"
          @keyup.enter="addTodo"
          placeholder="Add a new task"
          class="w-full p-2 rounded-l border"
        />
        <input
          v-model="newTodo.description"
          placeholder="Description"
          class="w-full p-2 rounded border ml-2"
        />
        <button @click="addTodo" class="bg-blue-500 text-white px-4 rounded-r">
          Add
        </button>
      </div>
      <ul>
        <li v-for="(todo, index) in todos" :key="index" class="flex items-center justify-between mb-2 p-2 bg-gray-50 rounded">
          <div class="flex items-center">
            <input type="checkbox" v-model="todo.completed" @change="updateCompletion(todo)" class="mr-2" />
            <span :class="{'line-through': todo.completed, 'text-gray-500': todo.completed}" @click="viewTodo(todo)">{{ todo.title }}</span>
          </div>
          <div class="flex items-center">
            <button @click="editTodoForm(todo)">Edit</button>
            <button @click="removeTodo(todo.id)" class="ml-2 text-red-500">Delete</button>
          </div>
        </li>
      </ul>

      <!-- Edit Form -->
      <div v-if="editingTodo" class="bg-gray-50 rounded p-2 mt-2">
        <h2 class="text-lg font-semibold mb-2">Edit Todo</h2>
        <input v-model="editingTodo.title" class="w-full p-2 rounded border mb-2" />
        <input v-model="editingTodo.description" class="w-full p-2 rounded border mb-2" placeholder="Description" />
        <input type="checkbox" v-model="editingTodo.completed" class="mr-2" />
        <button @click="editTodo" class="bg-green-500 text-white px-4 rounded">
          Save
        </button>
        <button @click="cancelEdit" class="bg-red-500 text-white px-4 rounded ml-2">
          Cancel
        </button>
      </div>

      <!-- Detailed View -->
      <div v-if="detailedTodo" class="bg-gray-50 rounded p-2 mt-2">
        <h2 class="text-lg font-semibold mb-2">Detailed View</h2>
        <p><strong>Title:</strong> {{ detailedTodo.title }}</p>
        <p><strong>Description:</strong> {{ detailedTodo.description }}</p>
        <p><strong>Created At:</strong> {{ detailedTodo.created_at }}</p>
        <p><strong>Completed:</strong> {{ detailedTodo.completed ? 'Yes' : 'No' }}</p>
        <button @click="closeDetailedView" class="bg-blue-500 text-white px-4 rounded mt-2">
          Close
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      newTodo: {
        title: '',
        description: '',
        completed: false,
      },
      todos: [],
      editingTodo: null,
      detailedTodo: null,
    };
  },
  mounted() {
    this.fetchTodos();
  },
  methods: {
    async fetchTodos() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/api/todos/');
        this.todos = response.data;
      } catch (error) {
        console.error('Error fetching todos:', error);
      }
    },
    async addTodo() {
      if (this.newTodo.title.trim() === '') return;

      try {
        const response = await axios.post('http://127.0.0.1:8000/api/todos/', this.newTodo);
        this.todos.push(response.data);
        this.newTodo = {
          title: '',
          description: '',
          completed: false,
        };
      } catch (error) {
        console.error('Error adding todo:', error);
      }
    },
    async removeTodo(todoId) {
      try {
        await axios.delete(`http://127.0.0.1:8000/api/todos/${todoId}/`);
        const index = this.todos.findIndex(todo => todo.id === todoId);
        if (index !== -1) {
          this.todos.splice(index, 1);
        }
      } catch (error) {
        console.error('Error deleting todo:', error);
      }
    },
    editTodoForm(todo) {
      this.editingTodo = { ...todo };
    },
    viewTodo(todo) {
      this.detailedTodo = { ...todo };
    },
    cancelEdit() {
      this.editingTodo = null;
    },
    async editTodo() {
      if (this.editingTodo) {
        const todoId = this.editingTodo.id;

        try {
          await axios.put(`http://127.0.0.1:8000/api/todos/${todoId}/`, this.editingTodo);
          const index = this.todos.findIndex(todo => todo.id === todoId);
          if (index !== -1) {
            this.todos[index] = { ...this.editingTodo };
            this.editingTodo = null;
          }
        } catch (error) {
          console.error('Error editing todo:', error);
        }
      }
    },
    closeDetailedView() {
      this.detailedTodo = null;
    },
    async updateCompletion(todo) {
      try {
        const todoId = todo.id;
        await axios.put(`http://127.0.0.1:8000/api/todos/${todoId}/`, todo);
      } catch (error) {
        console.error('Error updating todo completion:', error);
      }
    },
  },
};
</script>
