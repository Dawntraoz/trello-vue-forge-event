<script lang="ts" setup>
import { ref, toRefs } from "vue";
import type { Task } from "@/types";
import { v4 as uuidv4 } from "uuid";
import { useAlerts } from "@/stores/alerts";

const alerts = useAlerts();

const props = defineProps({
  id: String,
});
const { id: boardId } = toRefs(props);

const board = ref({
  id: boardId?.value || "1",
  title: "Let's have an amazing time at Vue.js forge!! 🍍",
  order: JSON.stringify([
    { id: "1", title: "backlog 🌴", taskIds: ["1", "2"] },
  ]),
});

const tasks = ref<Partial<Task>[]>([
  { id: "1", title: "Code like mad people!" },
  { id: "2", title: "Push clean code" },
]);

const addTask = async (task: Task) => {
  return new Promise((resolve, reject) => {
    const taskWithTheId = {
      ...task,
      id: uuidv4(),
    };
    tasks.value.push(taskWithTheId);
    resolve(taskWithTheId);
  });
};

const updateBoard = (b) => {
  board.value = b;
  alerts.success("Board updated!");
};

const deleteBoardIfConfirmed = () => {
  console.log("delete board");
};
</script>

<template>
  <AppPageHeading>{{ board.title }}</AppPageHeading>
  <BoardMenu :board="board" @deleteBoard="deleteBoardIfConfirmed" />
  <BoardDragAndDrop
    :tasks="tasks"
    :board="board"
    @update="updateBoard"
    :addTask="addTask"
  />
</template>
