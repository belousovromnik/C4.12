<template>
  <div class="container my-4">
    <div class="col-sm-10">
      <h1>Задачи - выполнено {{ made }} - невыполнено {{ not_made }}</h1>
      <confirmation :message="confirmationMessage"
                    v-model="showConfirmation"
                    v-if="showConfirmation">
      </confirmation>
      <div class="my-3">
        <button
          type="button"
          id="task-add"
          class="btn btn-success btn-sm align-left d-block"
          v-b-modal.todo-modal
        >
          Добавить задачу
        </button>
        <button
          type="button"
          id="clear-loc-st"
          class="btn btn-success btn-sm align-left d-block"
          @click="clearLocalStorage"
        >
          Очистить localStorage
        </button>
      </div>

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
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button
                  type="button"
                  class="btn btn-secondary btn-sm"
                  v-b-modal.todo-update-modal
                  @click="updateTodo(todo)"
                >
                  Обновить
                </button>
                &nbsp;
                <button type="button" class="btn btn-danger btn-sm" @click="deleteTodo(todo)">
                  X
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <b-modal ref="addTodoModal" id="todo-modal" title="Добавить задачу" hide-footer>
        <b-form @submit="onSubmit" @reset="onReset" class="w-100">
          <b-form-group
            id="form-description-group"
            label="Описание:"
            label-for="form-description-input"
          >
            <b-form-input
              id="form-description-input"
              type="text"
              v-model="addTodoForm.description"
              required
              placeholder="Завести задачу"
            ></b-form-input>
          </b-form-group>
          <b-form-group id="form-complete-group">
            <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button type="submit" variant="primary">Добавить</b-button>
          <b-button type="reset" variant="danger">Сброс</b-button>
        </b-form>
      </b-modal>

      <b-modal ref="updateTodoModal" id="todo-update-modal" title="Update" hide-footer>
        <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
          <b-form-group
            id="form-update-description-group"
            label="Описание:"
            label-for="form-update-description-input"
          >
            <b-form-input
              id="form-update-description-input"
              type="text"
              v-model="updateTodoForm.description"
              required
            >
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-update-complete-group">
            <b-form-checkbox-group v-model="updateTodoForm.is_completed" id="form-update-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button-group>
            <b-button type="submit" variant="primary">Обновить</b-button>
            <b-button type="reset" variant="danger">Сброс</b-button>
          </b-button-group>
        </b-form>
      </b-modal>
    </div>
  </div>
</template>

<script>
import Confirmation from './Confirmation.vue';

// import axios from 'axios';

// const todoListURL = 'http://localhost:5000/api/tasks/';
// const dataURL = todoListURL;
// const todoAddURL = 'http://localhost:5000/api/add-task/';

export default {
  name: 'Todo',
  data() {
    return {
      not_made: 0,
      made: 0,
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
      confirmationMessage: '',
      showConfirmation: false,
    };
  },
  methods: {
    saveIntolocalStorage() {
      // console.log(this.todos);
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    calcTodos() {
      let isNotCompleted = 0;
      let isCompleted = 0;
      if (this.todos.length > 0) {
        const arr = Object.values(this.todos);
        for (let i = 0; i < arr.length; i += 1) {
          if (arr[i].is_completed === true) {
            isCompleted += 1;
          } else {
            isNotCompleted += 1;
          }
        }
      }
      this.made = isCompleted;
      this.not_made = isNotCompleted;
    },
    getTodos() {
      if (localStorage.getItem('todos')) {
        try {
          this.todos = JSON.parse(localStorage.getItem('todos'));
          this.calcTodos();
        } catch (e) {
          localStorage.removeItem('todos');
        }
      }
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },

    clearLocalStorage() {
      localStorage.removeItem('todos');
      this.todos = [];
      this.getTodos();
    },

    onSubmit(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();

      const { description } = this.addTodoForm;
      /* eslint-disable camelcase */

      let uid_loc = 0;
      if (this.todos.length > 0) {
        const arr = Object.values(this.todos);
        uid_loc = arr[0].uid;
        for (let i = 0; i < arr.length; i += 1) {
          if (uid_loc < arr[i].uid) {
            uid_loc = arr[i].uid;
          }
        }
      }
      uid_loc += 1;

      if (typeof this.addTodoForm.is_completed[0] === 'undefined') {
        this.addTodoForm.is_completed = false;
      } else if (typeof this.addTodoForm.is_completed[0] === 'string') {
        if (this.addTodoForm.is_completed[0] === 'true') {
          this.addTodoForm.is_completed = true;
        } else {
          this.addTodoForm.is_completed = false;
        }
      }

      const { is_completed } = this.addTodoForm;

      const uid = uid_loc;
      this.todos.push({
        uid,
        description,
        is_completed,
      });

      this.confirmationMessage = 'Задача добавлена';
      this.showConfirmation = true;
      this.calcTodos();
      this.saveIntolocalStorage();
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.resetForm();
    },
    updateTodo(todo) {
      this.updateTodoForm = todo;
    },
    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.confirmationMessage = 'Задача обновлена';
      this.showConfirmation = true;

      if (typeof this.updateTodoForm.is_completed[0] === 'undefined') {
        this.updateTodoForm.is_completed = false;
      } else if (typeof this.updateTodoForm.is_completed[0] === 'string') {
        if (this.updateTodoForm.is_completed[0] === 'true') {
          this.updateTodoForm.is_completed = true;
        } else {
          this.updateTodoForm.is_completed = false;
        }
      }
      this.calcTodos();
      this.saveIntolocalStorage();
    },
    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.resetForm();
    },
    deleteTodo(todo) {
      // const todoURL = dataURL + todo.uid;
      this.confirmationMessage = `Задача удалена из списка ${todo.uid}`;
      this.showConfirmation = true;

      const toDo = this.todos.findIndex(item => item.uid === todo.uid);
      this.todos.splice(toDo, 1);
      this.calcTodos();
      this.saveIntolocalStorage();
    },
  },
  created() {
    this.getTodos();
  },
  components: {
    confirmation: Confirmation,
  },
};
</script>

<style>
button#task-add {
  margin-top: 20px;
  margin-bottom: 20px;
}
h1,
td {
  text-align: left;
}
.todo-uid {
  text-align: right;
}
</style>
