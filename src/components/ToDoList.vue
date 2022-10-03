<template>
    <section class="todoapp">
        <header class="header">
            <h1>待辦清單列表</h1>
            <input class="new-todo" autofocus autocomplete="off" placeholder="你有什麼需要完成的事情?" v-model="newTodo"
                @keyup.enter="addTodo" />
        </header>
        <section class="main" v-cloak>
            <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone" />
            <label for="toggle-all"></label>
            <ul class="todo-list">
                <li :class="{ completed: missioncatch.success }">
                    <div class=" view">
                        <div class="view">
                            <label>每日任務：完成Poker遊戲配對2次</label>
                        </div>
                    </div>
                </li>
                <li v-for="todo in filteredTodos" class="todo" :key="todo.id"
                    :class="{ completed: todo.completed, editing: todo == editedTodo }">
                    <div class="view" >
                        <input class=" toggle" type="checkbox" v-model="todo.completed" />
                        <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                        <button class="destroy" @click="removeTodo(todo)"></button>
                    </div>
                    <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo == editedTodo"
                        @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)" />
                </li>
            </ul>
        </section>
        <footer class="footer" v-show="todos.length" v-cloak>
            <span class="todo-count">
                <strong>{{ remaining }}</strong> {{ pluralize(remaining) }} left
            </span>
            <ul class="filters">
                <li>
                    <a href="#/all" :class="{ selected: visibility == 'all' }">全部</a>
                </li>
                <li>
                    <a href="#/active" :class="{ selected: visibility == 'active' }">待完成</a>
                </li>
                <li>
                    <a href="#/completed" :class="{ selected: visibility == 'completed' }">已完成</a>
                </li>
            </ul>
            <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
                清除完成事項
            </button>
        </footer>
    </section>
</template>
<script setup>
import { ref, computed, onMounted, watch, inject } from 'vue';
// localStorage persistence
const STORAGE_KEY = "todos-vuejs-3.0";
const todoStorage = {
    fetch: ()=>{
        
        const todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
        todos.forEach((todo, index) => {
           todo.id = index;
        });
        todoStorage.uid = todos.length;
        return todos;
    },
    save: (todos) => {
        return localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
        
    }
};
const MISSION_KEY = "mission-vuejs-3.0";
const missionStorage = {
    fetch: () => {
        const mission = JSON.parse(localStorage.getItem(MISSION_KEY));
        if(mission == null){
            const mission = JSON.parse(`{\"success\":false,\"time\":0}`)
            return mission;
        }
       
        return mission;
        
    },
    save: (missioncatch) => {
        return localStorage.setItem(MISSION_KEY, JSON.stringify(missioncatch));

    }
};

// visibility filters
const filters = {
    all: (todos) => todos,
    active: (todos) => todos.filter((todo) => !todo.completed),
    completed: (todos) => todos.filter((todo) => todo.completed),
}

const todos = ref([]);
const visibility = ref('all');
const filteredTodos = computed(() => {
    return filters[visibility.value](todos.value)
});
const remaining = computed(() => {
    return filters.active(todos.value).length;
});
const pluralize = (n) => {
    return n === 1 ? "item" : "items";
}
const newTodo = ref('');
const addTodo = () => {
    var value = newTodo.value && newTodo.value.trim();
    if (!value) {
        return;
    }
    todos.value.push({
        id: todos.value.length + 1,
        title: value,
        completed: false
    });
    newTodo.value = "";
}
const removeTodo = (todo) => {
    todos.value.splice(todos.value.indexOf(todo), 1);
}
const beforeEditCache = ref('')
const editedTodo = ref('')
const editTodo = (todo) => {
    beforeEditCache.value = todo.title
    editedTodo.value = todo
    
}
const doneEdit = (todo) => {
    if (!editedTodo.value) {
        return;
    }
    editedTodo.value = null;
    todo.title = todo.title.trim();
    if (!todo.title) {
        removeTodo(todo);
    }
}
const cancelEdit = (todo) => {
    editedTodo.value = null;
    todo.title = beforeEditCache.value;
}

const allDone = computed({
    get(){
        return remaining.value === 0;
    },
    set(value){
        todos.value.forEach((todo) => {
        todo.completed = value;
        });
    }
});
const onHashChange = () => {
    const hash = window.location.hash.replace(/#\/?/, "");
    if (filters[hash]) {
        visibility.value = hash
    } else {
        window.location.hash = "";
        visibility.value = 'all'
    }
}

const removeCompleted = () => {
    todos.value = filters.active(todos.value);
}

const vTodoFocus = {
    updated: (el, binding) => {
        if (binding.value) {
            el.focus();
            el.classList.add("focus"); 
        }else{
            el.blur();
            el.classList.remove("focus"); 
        }
    }
}
//定義每日任務參數
const missioncatch = ref({
    success: false,
    time: 0,
})
//接收emitter方法
const emitter = inject("emitter")
emitter.on('missionvalue', (data) => {
    missioncatch.value.success = data
    //抓完成的『日』00:00的毫秒
    const date = new Date().toLocaleDateString(); //抓到今天的日
    missioncatch.value.time = new Date(date).getTime() //抓到今天的日的毫秒
})

//check mission refresh
const checkDate = () => {
    const today = new Date();
    let milliseconds_Time_day = Math.trunc((today.getTime() - missioncatch.value.time) / (1000 * 3600 * 24)); //比完成日的00:00多一天
    if (milliseconds_Time_day >= 1){
        missioncatch.value.success = false
        console.log('每日任務重置')
    }
}

onMounted(
    () =>  {
        console.log('onMounted...')
        todos.value = todoStorage.fetch()
        window.addEventListener('hashchange', onHashChange)
        onHashChange()
        missioncatch.value = missionStorage.fetch()
        checkDate()
        console.log(localStorage)
        console.log(missioncatch.value)
        
    }
)
watch(
    todos,
    () => {
        console.log('watch todos...')
        todoStorage.save(todos.value);
    },
    {deep: true}
)
watch(
    missioncatch,
    () => {
        console.log('watch missioncatch...')
        missionStorage.save(missioncatch.value);
        console.log(localStorage)
    },
    { deep: true }
)


</script>
<style>
        [v-cloak] {
            display: none;
        }
        .focus{
            color: rgba(175, 47, 47, 1)
        }
        .todoapp h1{
            font-size: 70px;
            top: -135px;
            color: rgb(222 10 82);
        }
        .filters li a.selected {
            color: rgba(175, 47, 47, .5);
            border-color: rgba(175, 47, 47, .5);
        }
        .todo-list li label {
            background-image: url(data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23ededed%22%20stroke-width%3D%223%22/%3E%3C/svg%3E);
            background-repeat: no-repeat;
            background-position: center left;
        }
        .todo-list li.completed label {
            background-image: url(data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23bddad5%22%20stroke-width%3D%223%22/%3E%3Cpath%20fill%3D%22%235dc2af%22%20d%3D%22M72%2025L42%2071%2027%2056l-4%204%2020%2020%2034-52z%22/%3E%3C/svg%3E);
        }
</style>
