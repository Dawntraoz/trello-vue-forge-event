<script lang="ts" setup>
import { ref, toRefs } from "vue";
import { useAlerts } from "@/stores/alerts";

const alerts = useAlerts();

const props = defineProps({
  id: String,
});
const { id: boardId } = toRefs(props);

const board = ref({
  id: boardId.value,
  title: "Let's have an amazing time at Vue.js forge!! 🍍",
  order: JSON.stringify([
    { id: "1", title: "backlog 🌴", taskIds: ["1", "2"] },
  ]),
});

const tasks = ref([
  { id: "1", title: "Code like mad people!" },
  { id: "2", title: "Push clean code" },
]);

const updateBoard = (b) => {
  board.value = b;
  alerts.success("Board updated!");
};
</script>

<template>
  <AppPageHeading>{{ board.title }}</AppPageHeading>

  <BoardDragAndDrop :tasks="tasks" :board="board" @update="updateBoard" />
</template>
