<template>
  <Snake />
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component';
import Snake from './components/Snake.vue';

@Options({
  components: {
    Snake,
  },
})
export default class App extends Vue {}
</script>
