<script setup lang="ts">
import { useAlerts } from "@/stores/alerts";
import type { Board } from "@/types";
import { ref } from "vue";

import boardsQuery from "@/graphql/queries/boards.query.gql";
import createBoardMutation from "@/graphql/mutations/createBoard.mutation.gql";
import { useMutation, useQuery } from "@vue/apollo-composable";
import { computed } from "vue";

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

const newBoardPayload = {
  data: {
    title: "Test board 2",
  },
};
</script>

<template>
  <AppPageHeading>Boards</AppPageHeading>
  <div
    class="grid grid-cols-[20rem,20rem,20rem,20rem] overflow-auto gap-4 pb-4"
  >
    <BoardCard v-for="board in boards" :key="board.id" :board="board" />
    <button class="text-gray-500" @click="createBoard(newBoardPayload)">
      <span>New Board +</span>
    </button>
  </div>
  <p v-if="loading">Loading...</p>
</template>
