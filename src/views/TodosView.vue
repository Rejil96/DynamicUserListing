<script setup>
import { uid } from "uid";
import {
  ElButton,
  ElContainer,
  ElMain,
  ElHeader,
  ElRow,
  ElCol,
  ElInput,
  ElCheckbox,
} from "element-plus";

import { useThemeStore } from "../store/theme.js";
import { storeToRefs } from "pinia";

import { Delete, Edit, Check } from "@element-plus/icons-vue";

import Sidebar from "../components/Sidebar/Sidebar.vue";
import Header from "../components/Header/Header.vue";

import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";

const theme = useThemeStore();
const { darkTheme } = storeToRefs(theme);

const response = ref("");
const route = useRoute();

//Finding the current user based on path params
const currentPath = route.path.split("/").pop();
const userId = route.params.id;
const allUsers = ref([]);
const currentUserData = ref([]);

//states for creating todos
const userBasedTodos = ref([]);
const todoText = ref("");
const isTodoCompleted = ref(false);
const todoList = ref([]);
const isEditable = ref(false);
const editedInputValue = ref("");

onMounted(() => {
  onFetchUserData();
  isDataFoundInLocalStorage();
});

//This function will load the local storage data if its already exist
const isDataFoundInLocalStorage = () => {
  if (localStorage.getItem("userTodos")) {
    const getCurrentStoredItems = JSON.parse(localStorage.getItem("userTodos"));
    const replaceTodosFromLocalStorage = getCurrentStoredItems.filter(
      (eachData) => eachData.userId == userId
    )[0];
    if (replaceTodosFromLocalStorage) {
      todoList.value = replaceTodosFromLocalStorage.todos;
    } else {
      userBasedTodo();
    }
  }
};

//This function will load the currnt user based on the current path id
const onFetchUserData = async () => {
  response.value = await fetch("https://panorbit.in/api/users.json");
  allUsers.value = await response.value.json();
  currentUserData.value = allUsers.value.users.filter(
    (eachData) => eachData.id == userId
  )[0];
};

// This function will add the users in to the local storage along with the created todos list
const userBasedTodo = () => {
  if (localStorage.getItem("userTodos")) {
    const getCurrentStoredUsers = JSON.parse(localStorage.getItem("userTodos"));
    const isUserAltreadyExistInLs = getCurrentStoredUsers.filter(
      (eachData) => eachData.userId == userId
    )[0];
    if (isUserAltreadyExistInLs) {
      console.log("entering");
      const getCurrentStoredItems = JSON.parse(localStorage.getItem("userTodos"));
      const modifyUserTodos = getCurrentStoredItems.map((eachData) => {
        if (eachData.userId == userId) {
          return {
            userId: userId,
            todos: [...todoList.value],
          };
        }
        return eachData;
      });

      localStorage.setItem("userTodos", JSON.stringify(modifyUserTodos));
    } else {
      getCurrentStoredUsers.push({ userId, todos: todoList.value });
      localStorage.setItem("userTodos", JSON.stringify(getCurrentStoredUsers));
    }
  } else {
    const lsData = JSON.stringify([
      {
        userId,
        todos: todoList.value,
      },
    ]);
    localStorage.setItem("userTodos", lsData);
  }
};

const onAddTodo = () => {
  todoList.value.push({
    //16 is the length of generated unique id
    id: uid(16),
    todoText: todoText.value,
    isTodoCompleted: isTodoCompleted.value,
    isEditable: isEditable.value,
  });

  todoText.value = "";
  //The delete modification will store in local storage
  userBasedTodo();
};

const onCheck = (todoId) => {
  todoList.value = todoList.value.map((eachData) => {
    if (eachData.id == todoId) {
      return { ...eachData, isTodoCompleted: !eachData.isTodoCompleted };
    }
    return eachData;
  });

  userBasedTodo();
};

const onDelete = (todoId) => {
  todoList.value = todoList.value.filter((eachData) => eachData.id !== todoId);
  userBasedTodo();
};

const onEdit = (todoId) => {
  todoList.value = todoList.value.map((eachData) => {
    if (eachData.id === todoId) {
      editedInputValue.value = eachData.todoText;
      return { ...eachData, isEditable: !eachData.isEditable };
    }
    return eachData;
  });
  userBasedTodo();
};

const onSaveEdit = (todoId) => {
  todoList.value = todoList.value.map((eachData) => {
    if (eachData.id === todoId) {
      return { ...eachData, todoText: editedInputValue.value, isEditable: false };
    }
    return eachData;
  });
  userBasedTodo();
};
</script>

<template>
  <div class="bg-wrapper-user-detail" :class="{ darkTheme: darkTheme }">
    <Sidebar :currentPath="currentPath" :userId="userId" class="custom-show" />
    <div class="user-details-container">
      <Header
        :currentUserData="currentUserData"
        :currentPath="currentPath"
        :userId="userId"
      />
      <div class="todos-bg-wrapper">
        <el-row type="flex" justify="start" class="input-control-container">
          <el-col :span="24">
            <div class="input-control-wrapper">
              <el-input
                v-model="todoText"
                placeholder="Enter Todo Task"
                class="input-control"
                :class="{ 'custom-theme-bg': darkTheme }"
              />
              <el-button
                type="primary"
                class="add-todo-btn"
                :disabled="todoText.length < 6"
                @click="onAddTodo"
                >+</el-button
              >
            </div>
          </el-col>
        </el-row>
        <el-row type="flex" justify="start" class="input-control-container">
          <el-col :span="24">
            <TransitionGroup
              name="list"
              tag="ul"
              class="todo-listing-container"
              v-if="todoList.length > 0"
            >
              <li v-for="todo in todoList" :key="todo.id" class="list-item">
                <el-input
                  v-model="editedInputValue"
                  placeholder="Enter Todo Task"
                  v-if="todo.isEditable"
                  :value="todo.todoText"
                  class="custom-edit-input"
                />
                <p
                  v-else
                  class="item-panel"
                  :class="todo.isTodoCompleted ? 'custom-completed-text' : ''"
                >
                  {{ todo.todoText }}
                </p>
                <div class="button-wrapper">
                  <el-button
                    type="primary"
                    v-if="todo.isEditable"
                    @click="onSaveEdit(todo.id)"
                    :disabled="editedInputValue.length < 6"
                    >Save</el-button
                  >
                  <el-button
                    v-else
                    type="primary"
                    :icon="Edit"
                    circle
                    class="mobile-btn"
                    @click="onEdit(todo.id)"
                  />
                  <el-button
                    type="danger"
                    :icon="Delete"
                    circle
                    @click="onDelete(todo.id)"
                    class="mobile-btn"
                  />
                  <el-button
                    type="success"
                    :icon="Check"
                    circle
                    @click="onCheck(todo.id)"
                    :class="todo.isTodoCompleted ? '' : 'check-box-custom'"
                    class="mobile-btn"
                  />
                </div>
              </li>
            </TransitionGroup>
            <div v-if="todoList.length === 0" class="empty-bg-container">
              <h1 class="todoEmptyText" :class="{ 'theme-empty-color': darkTheme }">
                Todos Empty
              </h1>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bg-wrapper-user-detail {
  width: 100vi;
  height: 100vh;
  display: flex;
  box-sizing: border-box;
}

@media screen and (max-width: 768px) {
  .bg-wrapper-user-detail {
    height: 100%;
  }
}

.info-panel {
  font-size: 78px;
  opacity: 0.2;
}

@media screen and (max-width: 768px) {
  .info-panel {
    font-size: 38px;
    opacity: 0.2;
  }
}

.posts-container {
  width: 100%;
  height: 100%;
  display: flex;
}

.user-details-container {
  width: 100%;
  height: 100%;
  margin: 0px;
  padding: 0px;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-sizing: border-box;
}

.input-control {
  width: 40vw;
  height: 60px;
  border-top-right-radius: 0px !important;
  border-bottom-right-radius: 0px !important;
  border: 0px !important;
}

@media screen and (max-width: 768px) {
  .input-control {
    width: 100%;
  }
}

.todo-listing-container {
  width: 90%;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  height: 62vh;
  padding-left: 0px;
  list-style: none;
}

@media screen and (max-width: 768px) {
  .todo-listing-container {
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    margin: 0px;
  }
}

.add-todo-btn {
  width: 80px;
  height: 60px;
  font-weight: 600;
  font-size: 24px;
  border-top-left-radius: 0px !important;
  border-bottom-left-radius: 0px !important;
}

.input-control-container {
  margin-top: 80px;
  width: 100%;
  margin-left: 64px;
}

@media screen and (max-width: 768px) {
  .input-control-container {
    margin-top: 0px;
    width: 100%;
    margin-left: 0px;
    padding: 10px 20px;
  }
}

.todos-bg-wrapper {
  width: 100%;
}

.list-item {
  width: 95%;
  background-color: #d9f8ee;
  margin-top: 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-left: 10px #3f7ee4 solid;
}

@media screen and (max-width: 764px) {
  .list-item {
    width: 100%;
    box-sizing: border-box;
    overflow: hidden;
  }
}

li:nth-child(even) {
  background-color: #ffffe0;
}

.button-wrapper {
  height: 100%;
  display: flex;
  align-items: center;
  width: 120px;
  justify-content: space-between;
  margin-right: 40px;
}

@media screen and (max-width: 768px) {
  .button-wrapper {
    margin-right: 20px;
  }
}

.check-box-custom {
  margin-left: 10px !important;
}

.check-box-custom {
  background-color: #ffffff;
}

.custom-edit-input {
  background-color: transparent !important;
  height: 40px;
  width: 80%;
  margin: 10px;
  background-color: black;
}

.custom-completed-text {
  text-decoration: line-through;
}

.item-panel {
  margin-left: 40px;
  word-wrap: break-word;
  line-height: 24px;
}

@media screen and (max-width: 768px) {
  .item-panel {
    font-size: 14px;
  }
}

.empty-bg-container {
  width: 100%;
  height: 50vh;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 34px;
  font-weight: 600;
  opacity: 0.2;
}
.darkTheme {
  background-color: black;
}

.todoEmptyText {
  font-size: 78px;
  opacity: 0.9;
}

@media screen and (max-width: 768px) {
  .todoEmptyText {
    font-size: 38px;
    opacity: 0.9;
  }
}

.theme-empty-color {
  font-size: 78px;
  color: #ffffff;
  opacity: 0.8;
}

@media screen and (max-width: 768px) {
  .theme-empty-color {
    font-size: 38px;
  }
}

.custom-theme-bg :deep(.el-input__inner) {
  background-color: #252424;
  color: #ffffff;
}

.custom-theme-bg :deep(.el-input__wrapper) {
  background-color: #252424;
  box-shadow: 0 0 0 0px;
}

/* animation  */

.list-move, /* apply transition to moving elements */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}

@media screen and (max-width: 768px) {
  .custom-show {
    display: none;
  }
}

@media screen and (max-width: 768px) {
  .input-control-wrapper {
    width: 100%;
    display: flex;
    margin-top: 20px;
  }
}

@media screen and (max-width: 768px) {
  .mobile-btn{
    width: 30px;
    height: 30px;
  }
}
</style>
