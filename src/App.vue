<template>
    <div class="wrapper">
        <header class="header">
            <h1>Vue 3 todos</h1>
        </header>
        <section class="main">
            <div class="input-wrapper">
                <div class="checkbox"
                     @click="allCompleted(state.todoList.some(c => c.status === 'pending'))"
                     :class="{ checked: !state.todoList.some(c => c.status === 'pending') }" />
                <input autofocus type="text" v-model="state.todo" @keyup.enter="addTodo" placeholder="需要做什么？" />
            </div>
            <ul v-show="state.todoList.length > 0">
                <li v-for="(todo, index) in state.realList"
                    :key="index"
                    :class="{ completed: todo.status === 'fulfilled' }">
                    <div class="checkbox"
                         @click="completed(todo.id, todo.status === 'pending')"
                         :class="{ checked: todo.status === 'fulfilled' }" />
                    <p>{{ todo.text }}</p>
                    <div class="del" @click="delTodo(todo.id)">╳️</div>
                </li>
            </ul>
            <footer v-show="state.todoList.length > 0" class="footer">
                <span class="total-count">剩余{{ state.todoList.filter(c => c.status === 'pending').length }}件</span>
                <div class="control">
                    <div class="btn"
                         v-for="tab in state.tabs"
                         :key="tab.key"
                         :class="{ selected: state.currentTab === tab.key }"
                         @click="state.currentTab = tab.key">{{ tab.val }}
                    </div>
                </div>
                <p v-if="state.todoList.filter(c => c.status === 'fulfilled').length > 0"
                   class="clear"
                   @click="delAllCompleted">清除已完成</p>
            </footer>
        </section>
    </div>
</template>

<script>
    import { reactive, watch, toRaw, computed } from 'vue';

    const TODOKEY = 'TODOKEY';

    export default {
        setup () {
            const state = reactive({
                todo: '',
                todoList: JSON.parse(window.localStorage.getItem(TODOKEY)) || [],
                realList: computed(() => {
                    if (state.currentTab !== 'all') {
                        return state.todoList.filter(c => c.status === state.currentTab);
                    }
                    return state.todoList;
                }),
                currentTab: 'all',
                tabs: [{ key: 'all', val: '全部' }, { key: 'pending', val: '未完成' }, { key: 'fulfilled', val: '已完成' },]
            });

            const addTodo = () => {
                if (state.todo.trim() === '') return;
                state.todoList.push({
                    id: Date.now().toString(),
                    text: state.todo,
                    status: 'pending'
                });
                state.todo = '';
            };

            watch(
                () => state.todoList,
                todoList => window.localStorage.setItem(TODOKEY, JSON.stringify(toRaw(todoList))),
                { lazy: true, deep: true }
            );

            const delTodo = id => {
                const index = state.todoList.findIndex(c => c.id === id);
                if (index >= 0) state.todoList.splice(index, 1);
            };

            const completed = (id, checked) => {
                const index = state.todoList.findIndex(c => c.id === id);
                if (index >= 0) state.todoList[index].status = checked ? 'fulfilled' : 'pending';
            };

            const delAllCompleted = () => {
                state.todoList = state.todoList.filter(c => c.status === 'pending');
            };

            const allCompleted = checked => {
                state.todoList.forEach(c => {
                    c.status = checked ? 'fulfilled' : 'pending';
                });
            };

            return {
                state,
                addTodo,
                delTodo,
                completed,
                delAllCompleted,
                allCompleted
            };
        }
    };
</script>

<style>
    * {
        padding: 0;
        margin: 0;
    }

    body {
        font: 14px 'Helvetica Neue', Helvetica, Arial, sans-serif;
        background: #f5f5f5;
        color: #4d4d4d;
        min-width: 230px;
        max-width: 550px;
        margin: 0 auto;
        font-weight: 300;
    }

    .wrapper {
        padding-bottom: 100px;
    }

    .checkbox {
        width: 40px;
        height: 40px;
        background-image: url("data:image/svg+xml,%0A%3Csvg xmlns='http://www.w3.org/2000/svg' width='40' height='40' viewBox='-10 -18 100 135'%3E%3Ccircle cx='50' cy='50' r='50' fill='none' stroke='%23ededed' stroke-width='3'/%3E%3C/svg%3E");
    }

    .checkbox.checked {
        background-image: url("data:image/svg+xml,%0A%3Csvg xmlns='http://www.w3.org/2000/svg' width='40' height='40' viewBox='-10 -18 100 135'%3E%3Ccircle cx='50' cy='50' r='50' fill='none' stroke='%23bddad5' stroke-width='3'/%3E%3Cpath fill='%235dc2af' d='M72 25L42 71 27 56l-4 4 20 20 34-52z'/%3E%3C/svg%3E");
    }

    .header {
        padding-top: 14px;
    }

    .header h1 {
        color: rgba(175, 47, 47, 0.15);
        font-size: 100px;
        font-weight: 100;
        text-align: center;
    }

    .main {
        box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2), 0 25px 50px 0 rgba(0, 0, 0, 0.1);
        background-color: #fff;
    }

    .main .input-wrapper {
        position: relative;
    }

    .main .input-wrapper .checkbox {
        position: absolute;
        left: 0;
        top: 12px;
    }

    .main input {
        padding: 16px 16px 16px 60px;
        border: none;
        background: rgba(0, 0, 0, 0.003);
        box-shadow: inset 0 -2px 1px rgba(0, 0, 0, 0.03);
        width: 100%;
        font-size: 24px;
        font-family: inherit;
        font-weight: inherit;
        color: inherit;
        line-height: 1.4em;
        outline: 0;
    }

    .main input::-webkit-input-placeholder {
        font-style: italic;
        font-weight: 300;
        color: #e6e6e6;
    }

    .main ul {
        border-top: 1px solid #e6e6e6;
    }

    .main ul, .main ul li {
        list-style: none;
    }

    .main ul li {
        display: flex;
        height: 58px;
        align-items: center;
        border-bottom: 1px solid #ededed;
    }

    .main ul li:last-child {
        border-bottom: none;
    }

    .main ul li p {
        flex: 1;
        padding-left: 14px;
        font-size: 24px;
    }

    .main ul li.completed p {
        color: #d9d9d9;
        text-decoration: line-through;
    }

    .main ul li .del {
        display: none;
        padding: 10px;
        margin-right: 10px;
        cursor: pointer;
    }

    .main ul li:hover .del {
        display: block;
    }

    .main .footer {
        display: flex;
        align-items: center;
        color: #777;
        padding: 10px 15px;
        height: 20px;
        text-align: center;
        border-top: 1px solid #e6e6e6;
    }

    .main .footer .control {
        flex: 1;
        display: flex;
        justify-content: center;
    }

    .main .footer .control .btn {
        cursor: pointer;
        color: inherit;
        margin: 3px;
        padding: 3px 7px;
        text-decoration: none;
        border: 1px solid transparent;
        border-radius: 3px;
    }

    .main .footer .control .btn.selected {
        border-color: rgba(175, 47, 47, 0.2);
    }

    .main .footer .clear {
        cursor: pointer;
    }

</style>
