<template>
  <div id="app" class="app">
    <h1 class="title">To-Do List</h1>
    <form @submit.prevent="addTodo" class="form">
      <input v-model="newTodo" placeholder="Add a new todo" @blur="validateInput" class="input">
      <button :disabled="!newTodo || hasError" class="button">Add</button>
   </form>
    <ul class="todo-list">
      <li v-for="(todo, index) in todos" :key="todo.id" class="todo-item">
        <input type="checkbox" v-model="todo.isComplete" @change="updateTodo(index)" class="checkbox">
        <div class="todo-content">
          <span :class="{ completed: todo.isComplete }" class="title">{{ todo.title }}</span>
          <span class="date">{{ formatDate(todo.createdAt) }}</span>
        </div>
        <div class="todo-actions">
          <button @click="removeTodo(index)" class="remove-button">Remove</button>
          <button @click="editTodo(index)" class="edit-button">Edit</button>
        </div>
      </li>
    </ul>
    <div v-if="isEditing" class="edit-form">
      <form @submit.prevent="updateEditingTodo" class="form">
        <input v-model="editingTodo.title" placeholder="Edit todo" class="input">
        <button type="submit" class="button">Update</button>
        <button @click="cancelEditing" class="button">Cancel</button>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'

export default {
  name: 'App',
  data() {
    return {
      newTodo: '',
      todos: [],
      hasError: false,
      isEditing: false,
      editingTodo: null,
      editingTodoIndex: null
    }
  },
  computed: {
    filteredTodos() {
      return this.todos.filter(todo => !todo.isComplete)
    }
  },
  created() {
    axios.get('http://localhost:3000/todos')
      .then(response => {
        this.todos = response.data
      })
      .catch(error => {
        console.error(error)
      })
  },
  methods: {
    addTodo() {
      if (!this.newTodo) {
        this.hasError = true
        return
      }

      axios.post('http://localhost:3000/todos', {
        title: this.newTodo,
        isComplete: false,
        createdAt: new Date()
      })
        .then(response => {
          this.todos.push(response.data)
          this.newTodo = ''
          this.hasError = false
        })
        .catch(error => {
          console.error(error)
        })
    },
    removeTodo(index) {
      const todo = this.todos[index]
      axios.delete(`http://localhost:3000/todos/${todo.id}`)
        .then(() => {
          this.todos.splice(index, 1)
        })
        .catch(error => {
          console.error(error)
        })
    },
    updateTodo(index) {
      const todo = this.todos[index]
      axios.put(`http://localhost:3000/todos/${todo.id}`, {
        title: todo.title,
        isComplete: todo.isComplete,
        createdAt: todo.createdAt
})
        .then(() => {
          // Do nothing
        })
        .catch(error => {
          console.error(error)
        })
    },
    formatDate(date) {
      return moment(date).format('MMM D, YYYY h:mm A')
    },
    validateInput() {
      if (!this.newTodo) {
        this.hasError = true
      } else {
        this.hasError = false
      }
    },
    editTodo(index) {
      this.isEditing = true
      this.editingTodo = { ...this.todos[index] }
      this.editingTodoIndex = index
    },
    cancelEditing() {
      this.isEditing = false
      this.editingTodo = null
      this.editingTodoIndex = null
    },
    updateEditingTodo() {
      const todo = this.editingTodo

      axios.put(`http://localhost:3000/todos/${todo.id}`, {
        title: todo.title,
        isComplete: todo.isComplete,
        createdAt: todo.createdAt
      })
        .then(() => {
          this.todos[this.editingTodoIndex] = todo
          this.isEditing = false
          this.editingTodo = null
          this.editingTodoIndex = null
        })
        .catch(error => {
          console.error(error)
        })
    }
  }
}
</script>

<style>
.app {
  background: linear-gradient(to right, #ff9966, #ff5e62);
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}


.title {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

.form {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.input {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.title {
  flex: 1;
  margin-right: 10px;
}

.completed {
  text-decoration: line-through;
  color: #ccc;
}

.date {
  margin-top: 5px;
  color: #999;
}

.todo-list {
  list-style-type: none;
  padding: 0;
}

.todo-item {
  display: flex;
  align-items: center;
  border-bottom: 1px solid #ccc;
  padding: 10px 0;
  margin-bottom: 10px;
}

.todo-content {
  flex: 1;
}

.todo-actions {
  display: flex;
}

.remove-button {
  padding: 5px 10px;
  background-color: #ff5733;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-left: 10px;
}

.edit-button {
  padding: 5px 10px;
  background-color: #2196F3;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-left: 10px;
}

.edit-form {
  margin-top: 20px;
}
</style>