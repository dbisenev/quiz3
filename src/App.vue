<script setup lang="ts">
import { ref, computed, nextTick, watch } from "vue";

const tasks = ref<
    { id: number; title: string; priority: string; completed: boolean }[]
>([]);

const newTaskTitle = ref<string>("");
const newTaskPriority = ref<string>("low");

const addTask = async () => {
  if (newTaskTitle.value.trim()) {
    const newTask = {
      id: Date.now(),
      title: newTaskTitle.value,
      priority: newTaskPriority.value,
      completed: false,
    };
    tasks.value.push(newTask);
    newTaskTitle.value = "";
    newTaskPriority.value = "low";

    await nextTick(() => {
      const todoList = document.querySelector(".todo-list");
      todoList?.scrollTo(0, todoList.scrollHeight);
    });
  }
};

const toggleComplete = (id: number) => {
  const task = tasks.value.find((task) => task.id === id);
  if (task) task.completed = !task.completed;
};

const deleteTask = (id: number) => {
  tasks.value = tasks.value.filter((task) => task.id !== id);
};

const orderedTasks = computed(() =>
    tasks.value.slice().sort((a, b) => {
      const priorityOrder = { high: 1, medium: 2, low: 3 };
      return priorityOrder[a.priority] - priorityOrder[b.priority];
    })
);

const pendingCount = computed(() => tasks.value.filter((task) => !task.completed).length);

watch(
    () => tasks.value,
    (newTasks) => {
      console.log("Task list updated:", newTasks);
    },
    { deep: true }
);
</script>

<template>
  <div class="todo-app">
    <h1>To-Do List</h1>
    <p>Pending tasks: {{ pendingCount }}</p>

    <transition-group name="task" tag="div" class="todo-list">
      <div
          class="task"
          v-for="task in orderedTasks"
          :key="task.id"
          :class="{
            completed: task.completed,
            'high-priority': task.priority === 'high',
            'medium-priority': task.priority === 'medium'
          }"
      >
        <p>
          <strong>{{ task.title }}</strong> ({{ task.priority }})
        </p>
        <p>Status: {{ task.completed ? "Completed" : "Pending" }}</p>
        <button @click="toggleComplete(task.id)">Mark as {{ task.completed ? "Pending" : "Completed" }}</button>
        <button @click="deleteTask(task.id)">Delete</button>
      </div>
    </transition-group>

    <div class="add-task">
      <form @submit.prevent="addTask">
        <input
            v-model="newTaskTitle"
            type="text"
            placeholder="Task title"
            required
        />
        <select v-model="newTaskPriority">
          <option value="high">High</option>
          <option value="medium">Medium</option>
          <option value="low">Low</option>
        </select>
        <button type="submit">Add Task</button>
      </form>
    </div>
  </div>
</template>


<style scoped>
.todo-app {
  max-width: 500px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

.todo-list {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
}

.task {
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
  color: black;
  transition: all 0.5s ease;
}

.task.medium-priority {
  background-color: #fffccc;
}

.task.high-priority {
  background-color: #ffcccc;
}

.task.completed {
  text-decoration: line-through;
  color: gray;
}

/* Transition styles for tasks entering and leaving */
.task-enter-active,
.task-leave-active {
  transition: all 0.5s ease;
}

.task-enter-from {
  opacity: 0;
  transform: translateY(-40px);
}

.task-leave-to {
  opacity: 0;
  transform: translateY(40px);
}

.add-task {
  margin-top: 20px;
}

.add-task input,
.add-task select,
.add-task button {
  padding: 10px;
  margin: 5px;
  font-size: 14px;
}
</style>