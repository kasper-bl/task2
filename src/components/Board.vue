<template>
  <div class="board">
    <h1>Kanban Board</h1>
    <div class="columns">
      <Column
        v-for="col in state.columns"
        :key="col.id"
        :column="col"
        :state="state"
        :isFirstColumnFull="isFirstColumnFull"
      />
    </div>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';
import Column from './Column.vue';

export default {
  name: 'Board',
  components: {
    Column,
  },
  props: {
    state: Object,
  },
  setup(props) {
    const saved = localStorage.getItem('boardState');
    if (saved) {
      const parsed = JSON.parse(saved);
      props.state.columns[0].cards = parsed.columns[0].cards || [];
      props.state.columns[1].cards = parsed.columns[1].cards || [];
      props.state.columns[2].cards = parsed.columns[2].cards || [];
      props.state.nextId = parsed.nextId || 1;
    }

    watchEffect(() => {
      const toSave = {
        columns: props.state.columns.map(col => ({
          id: col.id,
          cards: col.cards.map(c => ({ ...c })),
        })),
        nextId: props.state.nextId,
      };
      localStorage.setItem('boardState', JSON.stringify(toSave));
    });

    const isFirstColumnFull = ref(false);

    return {
      isFirstColumnFull,
    };
  },
};
</script>

<style scoped>
.board {
  padding: 20px;
}
.columns {
  display: flex;
  gap: 20px;
}
</style>