<script setup lang="ts">
import { computed } from "vue";
import router from "@/router";
import { useAlerts } from "@/stores/alerts";
import { useMutation, useQuery } from "@vue/apollo-composable";

import boardsQuery from "@/graphql/queries/boards.query.gql";
import createBoardMutation from "@/graphql/mutations/createBoard.mutation.gql";

const { result, loading, onError } = useQuery(boardsQuery);
const boards = computed(() => result.value?.boardsList?.items || []);

const alerts = useAlerts();

onError(() => alerts.error("Error loading boards"));

const { mutate: createBoard } = useMutation(createBoardMutation, () => ({
  update(cache, { data: { boardCreate } }) {
    cache.updateQuery({ query: boardsQuery }, (res) => ({
      boardsList: {
        items: [...res.boardsList.items, boardCreate],
      },
    }));
  },
}));

const handleBoardCreate = async () => {
  const newBoardPayload = {
    data: {
      title: "My New Board",
    },
  };
  await createBoard(newBoardPayload);
  alerts.success("New Board created!");
};
</script>

<template>
  <AppPageHeading>Boards</AppPageHeading>
  <AppLoader v-if="loading" :overlay="true" />
  <div
    v-else
    class="grid grid-cols-[20rem,20rem,20rem,20rem,20rem] overflow-auto gap-4 pb-4"
  >
    <BoardCard v-for="board in boards" :key="board.id" :board="board" />
    <button class="text-gray-500" @click="handleBoardCreate">
      <span>New Board +</span>
    </button>
  </div>
</template>
