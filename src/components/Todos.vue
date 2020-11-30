<template>


  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <h2>Выполнено {{this.todos.filter(x => Boolean(x.is_completed)).length}}</h2>
      <h2>Невыполнено {{this.todos.length -
          this.todos.filter(x => Boolean(x.is_completed)).length}}</h2>
      <confirmation :message="confirmationMessage"
      v-if="showConfirmation" variant="danger"
      dismissible
      fade
      :show="showDismissibleAlert"></confirmation>
      <button type="button"
              id="task-add"
              class="btn btn-success btn-sm align-left d-block"
              v-b-modal.todo-modal
              @click="showDismissibleAlert=true"
              >
        Добавить задачу
      </button>
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
                <button type="button"
        class="btn btn-secondary btn-sm"
        v-b-modal.todo-update-modal
        @click="updateTodo(todo)"
        >
    Обновить
</button>
                &nbsp;
                <button type="button"
        class="btn btn-danger btn-sm"
        @click="deleteTodo(todo)">
  X
</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
</div>
<b-modal ref="addTodoModal"
         id="todo-modal"
         title="Добавить задачу"
         hide-footer>
  <b-form @submit="onSubmit" @reset="onReset" class="w-100">
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
    <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
      <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
    </b-form-checkbox-group>
    </b-form-group>
    <b-button type="submit" variant="primary">Добавить</b-button>
    <b-button type="reset" variant="danger">Сброс</b-button>
  </b-form>
</b-modal>
<b-modal ref="updateTodoModal"
         id="todo-update-modal"
         title="Update"
         hide-footer>
  <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
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
</template>

<style>
button#task-add {
  margin-top: 20px;
  margin-bottom: 20px;
}
h1, td {
  text-align: left;
}
.todo-uid {
  text-align: right;
}
</style>

<script>
import Confirmation from './Confirmation.vue';

export default {
  name: 'todoApp',
  data() {
    return {
      todos: [],
      todosCompleted: [],
      todosNotCompleted: [],
      todosTotal: [],
      addTodoForm: {
        uid: Number,
        description: '',
        is_completed: [],
      },
      updateTodoForm: {
        uid: Number,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '',
      showConfirmation: false,
      deleteBox: '',
      removeAllBox: '',
    };
  },
  components: {
    confirmation: Confirmation,
  },
  methods: {
    countTodos() {
      const todosToCount = this.todos;
      this.todosCompleted = [];
      this.todosNotCompleted = [];
      this.todosTotal = [];
      todosToCount.forEach((element) => {
        if (element.is_completed.length > 0) {
          this.todosCompleted.push(1);
          this.todosTotal.push(1);
        } else {
          this.todosNotCompleted.push(0);
          this.todosTotal.push(0);
        }
      });
    },
    getTodos() {
      localStorage.removeItem('loglevel:webpack-dev-server');
      const storageArray = Object.keys(localStorage);
      storageArray.sort((a, b) => a - b);
      this.todos = [];
      storageArray.forEach((key) => {
        const usedTodo = JSON.parse(localStorage.getItem(key));
        this.todos.push(usedTodo);
        localStorage.removeItem(key);
        usedTodo.uid = this.todos.length;
        localStorage.setItem(usedTodo.uid, JSON.stringify(usedTodo));
      });
      this.countTodos();
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },
    onSubmit(e) {
      e.preventDefault();
      this.$refs.addTodoModal.hide();
      const newTodo = {
        uid: this.todos.length + 1,
        description: this.addTodoForm.description,
        is_completed: this.addTodoForm.is_completed,
      };
      this.todos.push(newTodo);
      localStorage.setItem(Number(newTodo.uid), JSON.stringify(newTodo));
      this.confirmationMessage = `The Todo "${newTodo.description}" has been added`;
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
      this.resetForm();
    },
    onReset() {
      this.$refs.addTodoModal.hide();
      this.todos = [];
      this.getTodos();
    },
    updateTodo(todo) {
      this.updateTodoForm = todo;
    },
    onUpdateSubmit(e) {
      e.preventDefault();
      this.$refs.updateTodoModal.hide();
      localStorage.removeItem(this.updateTodoForm.uid);
      localStorage.setItem(Number(this.updateTodoForm.uid), JSON.stringify(this.updateTodoForm));
      this.confirmationMessage = `The Todo "${this.updateTodoForm.description}" has been updated`;
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
      this.resetForm();
    },
    onUpdateReset() {
      this.$refs.updateTodoModal.hide();
      this.todos = [];
      this.getTodos();
    },
    showDeleteBox(todo) {
      this.deleteBox = '';
      this.$bvModal.msgBoxConfirm(`Delete the Todo "${todo.description}"?`, {
        title: 'Please Confirm',
        size: 'm',
        buttonSize: 'm',
        okVariant: 'outline-danger',
        okTitle: 'YES',
        cancelVariant: 'outline-primary',
        cancelTitle: 'NO',
        footerClass: 'p-2',
        hideHeaderClose: false,
        headerBgVariant: 'danger',
        headerTextVariant: 'light',
        centered: true,
      })
        .then((value) => {
          if (value === true) {
            this.deleteTodo(todo);
          } else {
            this.resetForm();
          }
        });
    },
    deleteTodo(todo) {
      localStorage.removeItem(todo.uid);
      this.confirmationMessage = `The Todo "${todo.description}" has been removed`;
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
    },
    showRemoveAllBox() {
      if (this.todos.length > 0) {
        this.RemoveAllBox = '';
        this.$bvModal.msgBoxConfirm('Are you sure you want to remove all your Todos?', {
          title: 'Please Confirm',
          size: 'm',
          buttonSize: 'm',
          okVariant: 'outline-danger',
          okTitle: 'YES',
          cancelVariant: 'outline-primary',
          cancelTitle: 'NO',
          footerClass: 'p-2',
          hideHeaderClose: false,
          headerBgVariant: 'danger',
          headerTextVariant: 'light',
          centered: true,
        })
          .then((value) => {
            if (value === true) {
              this.removeAll();
            } else {
              this.resetForm();
            }
          });
      }
    },
    removeAll() {
      localStorage.clear();
      this.getTodos();
      this.confirmationMessage = 'All Todos have been removed';
      this.showConfirmation = true;
    },
  },
  created() {
    this.getTodos();
  },
};
</script>
