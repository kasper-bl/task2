<template>
  <div class="board">
    <h1>Todo</h1>
    <div class="columns">
      <Column
        v-for="col in state.columns"
        :key="col.id"
        :column="col"
        :state="state"
        :is-first-column-full="isFirstColumnFull"
        @progress-updated="handleProgressUpdate"
      />
    </div>
  </div>
</template>

<script>
import { ref, watchEffect, reactive } from 'vue';
import Column from './Column.vue';

export default {
  name: 'Board',
  components: {
    Column,
  },
  setup() {
    const state = reactive({
      columns: [
        { id: 'todo', title: 'В работе', maxCards: 3, cards: [] },
        { id: 'in-progress', title: 'На проверке', maxCards: 5, cards: [] },
        { id: 'done', title: 'Готово', maxCards: Infinity, cards: [] },
      ],
      nextId: 1,
    });

    const saved = localStorage.getItem('boardState');
    if (saved) {
      const parsed = JSON.parse(saved);
      state.columns[0].cards = parsed.columns[0].cards || [];
      state.columns[1].cards = parsed.columns[1].cards || [];
      state.columns[2].cards = parsed.columns[2].cards || [];
      state.nextId = parsed.nextId || 1;
    }

    watchEffect(() => {
      const toSave = {
        columns: state.columns.map(col => ({
          id: col.id,
          cards: col.cards.map(c => ({ ...c })),
        })),
        nextId: state.nextId,
      };
      localStorage.setItem('boardState', JSON.stringify(toSave));
    });

    const isFirstColumnFull = ref(false);

    const updateFirstColumnLock = () => {
      const todo = state.columns.find(c => c.id === 'todo');
      const inProgress = state.columns.find(c => c.id === 'in-progress');
      if (!todo || !inProgress) return;

      const isSecondFull = inProgress.cards.length >= inProgress.maxCards;
      const hasProgressOver50 = todo.cards.some(card => {
        const done = card.tasks.filter(t => t.completed).length;
        const total = card.tasks.length;
        return total > 0 && (done / total) * 100 > 50;
      });

      isFirstColumnFull.value = isSecondFull && hasProgressOver50;
    };

    const moveCardsAutomatically = () => {
      const allCards = [];
      for (const col of state.columns) {
        for (const card of col.cards) {
          allCards.push({ card, fromColumn: col.id });
        }
      }

      for (const { card, fromColumn } of allCards) {
        const total = card.tasks.length;
        const done = card.tasks.filter(t => t.completed).length;
        const progress = total ? Math.round((done / total) * 100) : 0;

        const sourceCol = state.columns.find(c => c.id === fromColumn);
        if (!sourceCol) continue;
        const idx = sourceCol.cards.indexOf(card);
        if (idx === -1) continue;

        if (fromColumn === 'todo' && progress > 50) {
          const targetCol = state.columns.find(c => c.id === 'in-progress');
          if (targetCol && targetCol.cards.length < targetCol.maxCards) {
            sourceCol.cards.splice(idx, 1);
            targetCol.cards.push(card);
          }
        } else if (fromColumn === 'in-progress' && progress === 100) {
          const targetCol = state.columns.find(c => c.id === 'done');
          if (targetCol) {
            sourceCol.cards.splice(idx, 1);
            targetCol.cards.push(card);
            card.completedAt = new Date().toLocaleString();
          }
        }
      }

      updateFirstColumnLock();
    };

    watchEffect(() => {
      state.columns.forEach(col =>
        col.cards.forEach(card =>
          card.tasks.forEach(() => {})
        )
      );
      moveCardsAutomatically();
    });

    setTimeout(moveCardsAutomatically, 0);

    return {
      state,
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