<script setup lang="ts">
import { Loader as KLoader } from "@progress/kendo-vue-indicators";

defineProps<{
  overlay: boolean;
}>();
</script>

<template>
  <div
    :class="{
      'absolute inset-0 flex justify-center items-center bg-white bg-opacity-60 z-1':
        overlay,
    }"
  >
    <KLoader type="pulsing" />
  </div>
</template>
