<script lang="ts" setup>
import { toRefs, computed, ref } from "vue";
import type { Task, Board } from "@/types";

import { useRouter } from "vue-router";
import { useAlerts } from "@/stores/alerts";

// GraphQL Apollo Client logic
import { useQuery, useMutation } from "@vue/apollo-composable";
import getBoardQuery from "@/graphql/queries/board.query.gql";
import boardsQuery from "@/graphql/queries/boards.query.gql";
import addTaskToBoardMutation from "@/graphql/mutations/addTaskToBoard.mutation.gql";
import updateBoardMutation from "@/graphql/mutations/updateBoard.mutation.gql";
import deleteBoardMutation from "@/graphql/mutations/deleteBoard.mutation.gql";

const alerts = useAlerts();
const router = useRouter();

const props = defineProps<{
  id: string;
}>();
const { id: boardId } = toRefs(props);

const {
  result: boardData,
  loading: loadingBoard,
  onError: onBoardError,
} = useQuery(
  getBoardQuery,
  { id: boardId.value },
  {
    fetchPolicy: "cache-and-network",
  }
);

onBoardError(() => alerts.error("Error loading board"));

const board = computed(() => boardData.value?.board || null);
const tasks = computed(() => board.value?.tasks?.items);

const { mutate: updateBoard, onDone: onBoardUpdated } =
  useMutation(updateBoardMutation);
onBoardUpdated(() => alerts.success("Board successfully updated!"));
const updateBoardTitle = (title: string) => {
  if (board.value.title === title) return;
  updateBoard({ id: boardId.value, title });
};

const { mutate: deleteBoard, onError: onErrorDeletingBoard } = useMutation(
  deleteBoardMutation,
  {
    update(cache) {
      cache.updateQuery({ query: boardsQuery }, (res) => ({
        boardsList: {
          items: res.boardsList.items.filter(
            (b: Board) => b.id !== boardId.value
          ),
        },
      }));
    },
  }
);
onErrorDeletingBoard(() => alerts.error("Error deleting board"));
const deleteBoardIfConfirmed = async () => {
  const yes = confirm("Are you sure you want to delete this board?");
  if (yes) {
    await deleteBoard({ id: boardId.value });
    router.push("/");
    alerts.success(`Board successfully deleted`);
  }
};

const {
  mutate: addTaskToBoard,
  onError: onErrorCreatingTask,
  onDone: onDoneCreatingTask,
} = useMutation(addTaskToBoardMutation);

let taskResolve = (task: Task) => {};
let taskReject = (message: Error) => {};

const addTask = (task: Task) => {
  return new Promise((resolve, reject) => {
    taskResolve = resolve;
    taskReject = reject;
    addTaskToBoard({
      boardId: boardId.value,
      ...task,
    });
  });
};

onErrorCreatingTask((error) => {
  taskReject(error);
  alerts.error("Error creating task");
});

onDoneCreatingTask((res) => {
  taskResolve(res.data.boardUpdate.tasks.items[0]);
  alerts.success("New task created!");
});
</script>

<template>
  <section v-if="board">
    <header class="flex justify-between">
      <AppPageHeading>
        <input
          @keydown.enter="($event.target as HTMLInputElement).blur()"
          @blur="updateBoardTitle(($event.target as HTMLInputElement).value)"
          type="text"
          :value="board.title"
        />
      </AppPageHeading>
      <BoardMenu :board="board" @deleteBoard="deleteBoardIfConfirmed" />
    </header>

    <BoardDragAndDrop
      :board="board"
      :tasks="tasks"
      @update="updateBoard"
      :addTask="addTask"
    />
  </section>
  <AppLoader v-if="loadingBoard" :overlay="true" />
</template>
