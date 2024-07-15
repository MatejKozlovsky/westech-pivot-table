<template>
  <v-icon
    v-if="selectedKey == sortingKey && selectedOrder == 'ASC'"
    icon="mdi-arrow-down-thin"
  >
  </v-icon>
  <v-icon
    v-if="selectedKey == sortingKey && selectedOrder == 'DESC'"
    icon="mdi-arrow-up-thin"
  >
  </v-icon>
</template>
<script setup lang="ts">
const props = defineProps<{
  selectedKey: String;
  selectedOrder: String;
  sortingKey: String;
}>();
</script>
