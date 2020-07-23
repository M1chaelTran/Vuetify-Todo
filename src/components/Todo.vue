<template>
  <v-container grid-list-md class="text-center md:p-8">
    <h1 class="text-4xl">Getting things done</h1>

    <div class="flex flex-col mt-8 mx-auto text-left md:w-4/5 relative">
      <v-text-field
        autofocus
        filled
        hide-details
        label="New todo item"
        v-model="newTodo"
        v-on:keyup.enter="handleAddTodo"
        :loading="adding"
      />
      <div :class="['border', 'border-t-0', {'rounded': !showCompleted}]" v-if="todos.length">
        <todo-list
          :items="todos"
          @itemClick="handleCompleteTodo"
          @itemDelete="handleDeleteTodo"
          class="rounded-b"
          :updating="updating"
          :deleting="deleting"
        />
      </div>
      <div class="border border-t-0 rounded-b" v-if="showCompleted && completed.length">
        <todo-list
          :items="completed"
          @itemClick="handleCompleteTodo"
          @itemDelete="handleDeleteTodo"
          class="rounded-b"
          :updating="updating"
          :deleting="deleting"
        />
      </div>

      <div class="text-right mt-1">
        <v-btn
          text
          @click="showCompleted = !showCompleted"
        >{{showCompleted?'Hide':'Show'}} completed</v-btn>
      </div>

      <v-overlay :value="fetching && !allTodos.length" absolute>
        <v-progress-circular indeterminate size="64" color="primary" />
      </v-overlay>
    </div>
  </v-container>
</template>

<script>
import { orderBy, findIndex } from "lodash";
import TodoList from "./TodoList.vue";

export default {
  components: {
    TodoList,
  },
  data() {
    return {
      newTodo: "",
      allTodos: [],
      error: "",
      showCompleted: false,
      fetching: true,
      adding: false,
      updating: false,
      deleting: false,
    };
  },
  async created() {
    try {
      this.fetching = true;
      const response = await fetch("/api/todos");
      const data = await response.json();
      this.allTodos = orderBy(data, ["updatedAt", "desc"]);
    } catch (error) {
      this.error = error.message;
    } finally {
      this.fetching = false;
    }
  },
  computed: {
    todos() {
      return this.allTodos.filter((x) => !x.complete);
    },
    completed() {
      return this.allTodos.filter((x) => x.complete);
    },
  },
  methods: {
    async handleAddTodo() {
      try {
        this.adding = true;
        const res = await fetch("/api/todos", {
          method: "post",
          body: JSON.stringify({ name: this.newTodo }),
        });
        const data = await res.json();
        this.allTodos = [data, ...this.allTodos];
        this.newTodo = "";
      } catch (error) {
        this.error = error.message;
      } finally {
        this.adding = false;
      }
    },

    async handleCompleteTodo(todo) {
      try {
        this.updating = todo.todoId;
        const res = await fetch(`/api/todos/${todo.todoId}`, {
          method: "patch",
          body: JSON.stringify({
            complete: !todo.complete,
            name: todo.name,
          }),
        });
        const data = await res.json();
        const todoIndex = findIndex(
          this.allTodos,
          (x) => x.todoId === todo.todoId
        );
        this.$set(this.allTodos, todoIndex, data);
      } catch (error) {
        this.error = error.mesage;
      } finally {
        this.updating = false;
      }
    },

    async handleDeleteTodo(todo) {
      try {
        this.deleting = todo.todoId;
        await fetch(`/api/todos/${todo.todoId}`, { method: "delete" });
        this.allTodos = this.allTodos.filter((x) => x.todoId !== todo.todoId);
      } catch (error) {
        this.error = error.mesage;
      } finally {
        this.deleting = false;
      }
    },
  },
};
</script>