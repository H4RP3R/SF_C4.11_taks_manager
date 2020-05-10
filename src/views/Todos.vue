<template>
<div class="container">
    <div class="col-sm-10">
        <h1 class="text-center">Задачи</h1>
        <p id="totl-tasks">Выполнено: {{ completed }} /
            Не выполнено: {{ notCompleted }}<br><span>(Всего задач: {{ total }})</span></p>
        <!-- Alert messqage -->
        <confirmation :message="message"
            :showConfirmation="showConfirmation"
            v-if="showConfirmation"
            v-on:disable="showConfirmation=false"></confirmation>
        <!-- Alert messqage -->

        <button type="
            button"
            id="task-add"
            v-b-modal.todo-modal
            class="btn btn-success btn-sm align-left d-block">Добавить задачу</button>

        <!-- Todos Table -->
        <table class="table table-dark table-stripped table-hover">
            <thead class="thead-light">
                <tr>
                    <th>Uid</th>
                    <th>Описание</th>
                    <th>Выполнена?</th>
                    <th></th>
                </tr>
            </thead>

            <tbody>
                <tr v-for="(todo, index) in todos"
                    :key="index">
                    <td class="todo-uid">{{ todo.uid }}</td>
                    <td>{{ todo.description }}</td>
                    <td>
                        <span v-if="todo.is_completed">Выполнено</span>
                        <span v-else>Не выполнено</span>
                    </td>
                    <td>
                        <div class="btn-group"
                            role="group">
                            <button type="button"
                                v-b-modal.todo-update-modal
                                @click="updateTodo(todo)"
                                class="btn btn-secondary btn-sm">Обновить</button>
                            &nbsp;
                            <button type="button"
                                @click="deleteTodo(todo)"
                                class="btn btn-danger btn-sm">X</button>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>
        <!-- Todos Table -->

    </div>

    <!-- Modal Window 1 -->
    <b-modal ref="addTodoModal"
        id="todo-modal"
        title="Добавить задачу"
        hide-footer>
        <b-form @submit="onSubmit"
            @reset="onReset"
            class="w-100">
            <b-form-group id="form-description-group"
                label="Описание:"
                label-for="form-description-input">
                <b-form-input id="form-description-input"
                    type="text"
                    v-model="addTodoForm.description"
                    required
                    placeholder="Завести задачу">
                </b-form-input>
            </b-form-group>
            <b-form-group id="form-complete-group">
                <b-form-checkbox-group v-model="addTodoForm.is_completed"
                    id="form-checks">
                    <b-form-checkbox>Задача выполнена?</b-form-checkbox>
                </b-form-checkbox-group>
            </b-form-group>
            <b-button type="submit"
                variant="primary">Добавить</b-button>
            <b-button type="reset"
                variant="danger">Сброс</b-button>
        </b-form>
    </b-modal>
    <!-- Modal Window 1 -->

    <!-- Modal Window 2 -->
    <b-modal ref="updateTodoModal"
        id="todo-update-modal"
        title="Update"
        hide-footer>
        <b-form @submit="onUpdateSubmit"
            @reset="onUpdateReset"
            class="w-100">
            <b-form-group id="form-update-description-group"
                label="Описание:"
                label-for="form-update-description-input">
                <b-form-input id="form-update-description-input"
                    type="text"
                    v-model="updateTodoForm.description"
                    required>
                </b-form-input>
            </b-form-group>
            <b-form-group id="form-update-complete-group">
                <b-form-checkbox-group v-model="updateTodoForm.is_completed"
                    id="form-update-checks">
                    <b-form-checkbox>Задача выполнена?</b-form-checkbox>
                </b-form-checkbox-group>
            </b-form-group>
            <b-button-group>
                <b-button type="submit"
                    variant="primary">Обновить</b-button>
                <b-button type="reset"
                    variant="danger">Сброс</b-button>
            </b-button-group>
        </b-form>
    </b-modal>
    <!-- Modal Window 2 -->

</div>
</template>


<script>
import Confirmation from '@/components/Confirmation.vue';


export default {
    name: 'Todo',
    data() {
        return {
            todos: [],
            addTodoForm: {
                description: '',
                is_completed: [],
            },
            updateTodoForm: {
                uid: 0,
                description: '',
                is_completed: [],
            },
            message: '',
            showConfirmation: false,
            defaultTodos: [{
                    'description': 'прочитать книгу',
                    'is_completed': false,
                    'uid': 0,
                },
                {
                    'description': 'учиться жонглировать 30 минут',
                    'is_completed': false,
                    'uid': 1,
                },
                {
                    'description': 'помыть посуду',
                    'is_completed': false,
                    'uid': 2,
                },
                {
                    'description': 'поесть',
                    'is_completed': false,
                    'uid': 3,
                },
            ]
        };
    },
    computed: {
        total: function() {
            return this.todos.length;
        },
        completed: function() {
            let n = 0;
            this.todos.forEach(elem => {
                if (elem.is_completed === true) {
                    n++
                }
            });
            return n;
        },
        notCompleted: function() {
            let n = 0;
            this.todos.forEach(elem => {
                if (elem.is_completed === false) {
                    n++
                }
            });
            return n;
        },
    },
    components: {
        confirmation: Confirmation,
    },
    methods: {
        getTodos() {
            if (!localStorage.getItem('todos')) {
                // use default tasks
                this.todos = this.defaultTodos;
                localStorage.setItem('todos', JSON.stringify(this.todos));
            } else {
                // use tasks from the local storage if the key exists
                this.todos = JSON.parse(localStorage.getItem('todos'));
            }
        },
        resetForm() {
            this.addTodoForm.description = '';
            this.addTodoForm.is_completed = [];
            this.updateTodoForm.description = '';
            this.updateTodoForm.is_completed = [];
        },
        onSubmit(event) {
            event.preventDefault();
            this.$refs.addTodoModal.hide();
            const taskToAdd = {
                description: this.addTodoForm.description,
                is_completed: (this.addTodoForm.is_completed.length > 0) ? true : false,
                uid: this.createUid(),
            }
            this.todos.push(taskToAdd)
            localStorage.setItem('todos', JSON.stringify(this.todos));
            this.message = `Задача "${taskToAdd.description}" добавлена`;
            this.showConfirmation = true;
            this.resetForm()
        },
        onReset(event) {
            event.preventDefault();
            this.$refs.addTodoModal.hide();
            this.resetForm();
        },
        updateTodo(todo) {
            // this.updateTodoForm = todo;
            this.updateTodoForm.description = todo.description;
            this.updateTodoForm.is_completed = (todo.is_completed) ? [true] : [false];
            this.updateTodoForm.uid = todo.uid;
        },
        onUpdateSubmit(event) {
            event.preventDefault();
            this.$refs.updateTodoModal.hide();
            const taskToUpdate = {
                description: this.updateTodoForm.description,
                is_completed: (this.updateTodoForm.is_completed.length > 1) ? true : false,
                uid: this.updateTodoForm.uid,
            }
            // change the task in todos
            this.todos.forEach(function(elem, index) {
                if (elem.uid === taskToUpdate.uid) {
                    this[index] = taskToUpdate;
                }
            }, this.todos)
            localStorage.setItem('todos', JSON.stringify(this.todos));
            this.getTodos();
            this.message = `Задача  "ID: ${taskToUpdate.uid}" обновлена`;
            this.showConfirmation = true;

        },
        onUpdateReset(event) {
            event.preventDefault();
            this.$refs.updateTodoModal.hide();
            this.resetForm();
        },
        deleteTodo(todo) {
            this.todos.forEach(function(elem, index) {
                if (elem.uid === todo.uid) {
                    this.splice(index, 1);
                }
            }, this.todos)
            localStorage.setItem('todos', JSON.stringify(this.todos));
        },
        createUid() {
            if (this.todos.length === 0) {
                return 0;
            }
            const lastId =
                this.todos.length > 0 ?
                this.todos[this.todos.length - 1].uid :
                0;
            let newUid = lastId + 1;
            let ids = [] // make an array with existing IDs
            this.todos.forEach(elem => {
                ids.push(elem.uid)
            })
            // if new ID already exists, increase its value
            while (ids.includes(newUid)) {
                newUid += 1;
            }
            return newUid;
        }
    },
    created() {
        this.getTodos();
    },
};
</script>

<style>
button#task-add {
    margin-top: 20px;
    margin-bottom: 20px;
}

#totl-tasks>span {
    font-style: italic;
}

h1,
td {
    text-align: left;
}

.todo-uid {
    text-align: right;
}
</style>
