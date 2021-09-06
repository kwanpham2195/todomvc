<template>
  <div v-if="loading">Loading...</div>
  <section class="todoapp" v-else>
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        autofocus
        autocomplete="off"
        placeholder="What needs to be done?"
        :value="newTodo"
        @change="setNewTodo"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main" v-show="todos.length > 0">
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          class="todo"
          :key="todo.id"
          :class="{
            completed: todo.completed,
            editing: editingTodo && todo.id === editingTodo.id,
          }"
        >
          <input
            v-if="editingTodo"
            class="edit"
            type="text"
            v-model="editingTodo.title"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit"
          />
          <div class="view">
            <input
              class="toggle"
              type="checkbox"
              :checked="todo.completed"
              @change="changeTodoCompleted(todo, $event)"
            />
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length > 0">
      <span class="todo-count">
        <strong>{{ remaining }}</strong
        >{{ remaining | pluralize }} left
      </span>
      <ul class="filters">
        <li>
          <a
            @click.prevent="visibility = 'all'"
            :class="{ selected: visibility == 'all' }"
          >
            All
          </a>
        </li>
        <li>
          <a
            @click.prevent="visibility = 'active'"
            :class="{ selected: visibility == 'active' }"
          >
            Active
          </a>
        </li>
        <li>
          <a
            @click.prevent="visibility = 'completed'"
            :class="{ selected: visibility == 'completed' }"
          >
            Completed
          </a>
        </li>
      </ul>
    </footer>
  </section>
</template>

<script>
const filters = {
  all: (todos) => {
    return todos;
  },
  active: (todos) => {
    return todos.filter((todo) => {
      return !todo.completed;
    });
  },
  completed: (todos) => {
    return todos.filter((todo) => {
      return todo.completed;
    });
  },
};
export default {
  name: "Home",
  data() {
    return {
      newTodo: "",
      visibility: "all",
      editingTodo: null,
    };
  },
  filters: {
    pluralize: function (n) {
      return n === 1 ? " item" : " items";
    },
  },
  created() {
    this.$store.dispatch("loadTodos");
  },
  computed: {
    loading() {
      return this.$store.state.loading;
    },
    todos() {
      return this.$store.state.todos;
    },
    filteredTodos() {
      return filters[this.visibility](this.todos);
    },
    remaining() {
      return filters.active(this.todos).length;
    },
  },
  methods: {
    setNewTodo(e) {
      this.newTodo = e.target.value;
    },
    async addTodo() {
      await this.$store.dispatch("addTodo", this.newTodo);
      this.newTodo = "";
    },
    removeTodo(todo) {
      this.$store.dispatch("removeTodo", todo);
    },
    changeTodoCompleted(todo, e) {
      this.$store.dispatch("updateTodo", {
        todo: todo,
        payload: {
          completed: e.target.checked,
        },
      });
    },
    editTodo(todo) {
      this.editingTodo = { ...todo };
    },
    doneEdit(todo) {
      this.$store.dispatch("updateTodo", {
        todo: todo,
        payload: {
          title: this.editingTodo.title,
        },
      });
      this.editingTodo = null;
    },
    cancelEdit() {
      this.editingTodo = null;
    },
  },
};
</script>
