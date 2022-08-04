<template>
    <section class="todoapp">
        <header class="header">
            <h1>todos</h1>
            <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo"
                @keyup.enter="addTodo" />
        </header>
        <section class="main" v-show="todos.length" v-cloak>
            <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone" />
            <label for="toggle-all"></label>
            <ul class="todo-list">
                <li v-for="todo in filteredTodos" class="todo" :key="todo.id"
                    :class="{ completed: todo.completed, editing: todo == editedTodo }">
                    <div class="view">
                        <input class="toggle" type="checkbox" v-model="todo.completed" />
                        <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                        <button class="destroy" @click="removeTodo(todo)"></button>
                    </div>
                    <input class="edit" type="text" v-model="todo.title" 
                        @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)" />
                </li>
            </ul>
        </section>
        <footer class="footer" v-show="todos.length" v-cloak>
            <span class="todo-count">
                <strong>{{ remaining }}</strong> {{ remaining | pluralize }} left
            </span>
            <ul class="filters">
                <li>
                    <a href="#/all" :class="{ selected: visibility == 'all' }">All</a>
                </li>
                <li>
                    <a href="#/active" :class="{ selected: visibility == 'active' }">Active</a>
                </li>
                <li>
                    <a href="#/completed" :class="{ selected: visibility == 'completed' }">Completed</a>
                </li>
            </ul>
            <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
                Clear completed
            </button>
        </footer>
    </section>
</template>
<script setup>
import { ref, computed } from 'vue';
// localStorage persistence
const STORAGE_KEY = "todos-vuejs-2.0";
const todoStorage = {
    fetch: function() {
        const todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
        todos.forEach((todo, index) => {
            todo.id = index;
        });
        todoStorage.uid = todos.length;
        return todos;
    },
    save: function(todos) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
    }
};
// visibility filters
const filters = {
    all: (todos) => todos,
    active: (todos) => todos.filter((todo) => !todo.completed),
    completed: (todos) => todos.filter((todo) => todo.completed),
}
const newTodo = ref('');
const todos = ref([]);
const visibility = ref('all');
const filteredTodos = computed(() => {
    filters[visibility.value](todos.value);
});
const remaining = computed(() => {
    filters.active(todos.value).length;
});
const pluralize = (n) => {
    n === 1 ? "item" : "items";
};
const addTodo = () => {
    var value = newTodo.value && newTodo.value.trim();
    if (!value) {
        return;
    }
    todos.value.push({
        id: todoStorage.uid++,
        title: value,
        completed: false
    });
    newTodo.value = "";  
    console.log(todos.value)
}
const removeTodo = (todo) => {
    todos.value.splice(todos.value.indexOf(todo), 1);
}

</script>
<style>
        [v-cloak] {
            display: none;
        }
</style>
