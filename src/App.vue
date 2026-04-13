<template>
  <div id="app">
    <Minesweeper />
  </div>
</template>

<script>
import Minesweeper from './components/Minesweeper.vue';

export default {
  name: 'App',
  components: {
    Minesweeper,
  },
};
</script>