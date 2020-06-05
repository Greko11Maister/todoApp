<template>
  <div>
    <h2 v-if="errorMessage.length > 0">{{errorMessage}}</h2>
    <section class="todoapp" v-cloak>
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo" >
      </header>
      <section class="main" v-show="todos.length">
        <ul class="todo-list">
          <li class="todo" v-for="todo in todos" :key="todo.id" :class="{completed: todo.completed, editing: todo == editedTodo}">
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input
              class="edit"
              type="text"
              v-model="todo.title"
              @keyup.enter="doneEdit(todo)"
              @blur="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)"
              v-todo-focus="todo == editedTodo"
            >
          </li>
        </ul>
      </section>
    </section>
  </div>
</template>

<script>
import "./assets/css/index.css";
import "./assets/css/app.css";
import Wails from '@wailsapp/runtime';

export default {
  name: "app",
  data() {
    return {
      newTodo: "",
      editedTodo: null,
      errorMessage: "",
      todos: [],
    }
  },
  watch: {
    todos: function(todos) {
      window.backend.saveList(JSON.stringify(todos, null, 2));
    },
    deep: true
  },
  directives: {
    "todo-focus": function(el, binding) {
      if (binding.value) {
        el.focus();
      }
    }
  },
  mounted() {
    window.backend.loadList().then((list) => {
      try {
        this.todos = JSON.parse(list);
      } catch (e) {
        
        Wails.Log.Info("An error was thrown: " + e.message);
        this.errorMessage = "Unable to load todo list";
        setTimeout(() => {
          this.errorMessage = "";
        }, 3000);
      }
    });
  },
  methods: {
    addTodo: function() {
      var value = this.newTodo && this.newTodo.trim();
      if (!value) {
        return;
      }
      this.todos.push({
        id: this.todos.length,
        title: value,
        completed: false
      });
      this.newTodo = "";
    },
    removeTodo: function(todo) {
      var index = this.todos.indexOf(todo);
      this.todos.splice(index, 1);

      for(var i=0; i<this.todos.length; i++){ 
        this.todos[i].id = i;
      }
    },
    editTodo: function(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },
    doneEdit: function(todo) {
      if (!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.removeTodo(todo);
      }
    },
    cancelEdit: function(todo) {
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    }
  }
};
</script>

<style>
h2 {
  text-align: center;
  color: white;
  background-color: red;
  min-width: 230px;
  max-width: 550px;
  padding: 1rem;
  border-radius: 0.5rem;
}
</style>