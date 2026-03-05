<template>
  <div id="app">
    <Board />
  </div>
</template>

<script>
import { reactive } from 'vue'
import Board from './components/Board.vue'

export default {
  name: 'App',
  components: {
    Board,
  },
}
</script>
