<script lang="ts" setup>
import { computed, toRefs } from "vue";

const props = defineProps<{
  src: string;
}>();

const { src } = toRefs(props);

const twicpicsURL = computed(() => {
  const url = toTwipicsURL(src.value);
  return url;
});

const toTwipicsURL = (url: string) => {
  return url.replace("https://cdn.filestackcontent.com/", "");
};
</script>

<template>
  <TwicImg :src="twicpicsURL" ratio="none" />
</template>
