<template>
  <button
    v-if="!showButton"
    @click="$emit('update:start')"
  >
    Start Game
  </button>
</template>

<script lang="ts">
export default {
  name: "Controls",
  props: {
    showButton: {
      type: Boolean,
      required: true
    }
  }
};
</script>
