<script setup>
import { ref, computed } from "vue";
import { v4 as uuid } from "uuid";

const newTodo = ref('')
const beforeEditCache = ref('')
const filter = ref('all')
const todos = ref([])

const remaining = computed(() => {
    return todos.value.filter(todo => !todo.completed).length
})

const anyRemaining = computed(() => {
    return remaining.value != 0
})

const todosFiltered = computed(() => {
    if (filter.value == 'all') {
        return todos.value
    } else if (filter.value == 'active') {
        return todos.value.filter(todo => !todo.completed)
    } else if (filter.value == 'completed') {
        return todos.value.filter(todo => todo.completed)
    }

    return todos.value
})

const showClearCompletedButton = computed(() => {
    return todos.value.filter(todo => todo.completed).length > 0
})

const vFocus = {
    mounted: (el) => el.focus()
}

const addTodo = () => {
    if (newTodo.value.trim().length == 0) {
        return
    }

    todos.value.push({
        id: uuid(),
        title: newTodo.value,
        completed: false,
        editing: false
    })

    newTodo.value = ''
}

const editTodo = (todo) => {
    beforeEditCache.value = todo.title
    todo.editing = true
}

const doneEdit = (todo) => {
    if (todo.title.trim() == '') {
        todo.title = beforeEditCache.value
    }

    todo.editing = false
}

const cancelEdit = (todo) => {
    todo.title = beforeEditCache.value
    todo.editing = false
}

const removeTodo = (index) => {
    todos.value.splice(index, 1)
}

const checkAllTodos = () => {
    todos.value.forEach(todo => todo.completed = event.target.checked)
}

const clearCompleted = () => {
    todos.value = todos.value.filter(todo => !todo.completed)
}
</script>

<template>
    <div class="list-wrapper">
        <input type="text" class="todo-input" placeholder="What needs to be done" v-model="newTodo"
            @keyup.enter="addTodo">
        <div class="todos-container">
            <transition-group name="fade" enter-active-class="animated fadeInUp"
                leave-active-class="animated fadeOutDown">
                <div class="todo-item" v-for="(todo, id) in todosFiltered" :key="todo.id">
                    <input type="checkbox" v-model="todo.completed">
                    <div v-if="!todo.editing" class="todo-item-label" :class="{ completed: todo.completed }"
                        @dblclick="editTodo(todo)">
                        {{ todo.title }}
                    </div>
                    <input v-else type="text" class="todo-item-edit" v-model="todo.title" @blur="doneEdit(todo)"
                        @keyup.esc="cancelEdit(todo)" v-focus>
                    <div class="remove-item" @click="removeTodo(id)">
                        &times;
                    </div>
                </div>
            </transition-group>
        </div>
        <div class="extra-container column">
            <div>
                <label>
                    <input type="checkbox" :checked="!anyRemaining" @change="checkAllTodos"> Check All
                </label>
            </div>
            <div>
                {{ remaining }} items left
            </div>
        </div>

        <div class="extra-container">
            <div>
                <button :class="{ active: filter == 'all' }" @click="filter = 'all'">
                    All
                </button>
                <button :class="{ active: filter == 'active' }" @click="filter = 'active'">
                    Active
                </button>
                <button :class="{ active: filter == 'completed' }" @click="filter = 'completed'">
                    Completed
                </button>
            </div>
        </div>

        <div>
            <transition name="fade">
                <button v-if="showClearCompletedButton" @click="clearCompleted">
                    Clear completed
                </button>
            </transition>
        </div>
    </div>
</template>

<style lang="scss">
@import url("https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css");

.todos-container {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    padding: 10px;

    @media (max-width: 768px) {
        display: flex;
        flex-direction: column;
        row-gap: 10px;
    }
}

.list-wrapper {
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;

    &:focus {
        outline: 0;
    }
}

.todo-item {
    display: flex;
    align-items: center;    
    animation-duration: 0.3s;

    margin: 10px 0;
}

.remove-item {
    cursor: pointer;
    margin-left: 20px;

    &:hover {
        color: black;
    }
}

/* .todo-item-left {
    display: flex;
    align-items: center;
} */

.todo-item-label {
    width: 80%;
    padding: 10px;
    border: 1px solid white;
    margin-left: 12px;
}

.todo-item-edit {
    font-size: 24px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    font-family: 'Avenir', Arial, Helvetica, sans-serif;

    &:focus {
        outline: none;
    }
}

.completed {
    text-decoration: line-through;
    color: grey;
}

.extra-container {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    font-size: 16px;
    border-top: 1px solid lightgrey;
    padding-top: 14px;
    margin-bottom: 14px;
}

.column {
    flex-direction: column;
}

button {
    font-size: 14px;
    border-color: white;
    cursor: pointer;
    padding: 10px;

    &:hover {
        background: lightgreen;
    }

    &:focus {
        outline: none;
    }
}

.active {
    background: lightgreen;
}

// CSS Transitions
.fade-enter-active,
fade-leave-active {
    transition: opacity .2s;
}

.fade-enter,
fade-leave-to {
    opacity: 0;
}
</style>