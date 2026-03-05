<template>
  <div class="card" :class="{ locked: isLocked }">
    <h3>{{ card.title }}</h3>
    <ul class="task-list">
      <TaskItem
        v-for="task in card.tasks"
        :key="task.id"
        :task="task"
        :isLocked="isLocked"
        @toggle="onTaskToggle"
      />
    </ul>
    <p>Прогресс: {{ progress }}%</p>
    <p v-if="card.completedAt">Завершена: {{ card.completedAt }}</p>
  </div>
</template>

<script>
import { computed } from 'vue';
import TaskItem from './TaskItem.vue';

export default {
  name: 'Card',
  components: {
    TaskItem,
  },
  props: {
    card: Object,
    state: Object,
    isLocked: Boolean,
  },
  emits: ['progress-changed'],
  setup(props, { emit }) {
    const progress = computed(() => {
      const total = props.card.tasks.length;
      const done = props.card.tasks.filter(t => t.completed).length;
      return total ? Math.round((done / total) * 100) : 0;
    });

    const getCurrentColumnIndex = () => {
      for (let i = 0; i < props.state.columns.length; i++) {
        const col = props.state.columns[i];
        if (col.cards.some(c => c.id === props.card.id)) {
          return i;
        }
      }
      return -1;
    };

    const moveCard = (fromIndex, toIndex) => {
      if (fromIndex === -1 || toIndex === -1) return;

      const fromCol = props.state.columns[fromIndex];
      const toCol = props.state.columns[toIndex];

      const cardIndex = fromCol.cards.findIndex(c => c.id === props.card.id);
      if (cardIndex === -1) return;

      const [card] = fromCol.cards.splice(cardIndex, 1);
      toCol.cards.push(card);
    };

    const onTaskToggle = () => {
      emit('progress-changed');

      const currentColIndex = getCurrentColumnIndex();

      if (currentColIndex === 0 && progress.value > 50) {
        moveCard(0, 1); 
      } else if (currentColIndex === 1 && progress.value === 100) {
        props.card.completedAt = new Date().toLocaleString();
        moveCard(1, 2); 
      }
    };

    return {
      progress,
      onTaskToggle,
    };
  },
};
</script>

<style scoped>
.card {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 10px;
  background-color: white;
}
.locked {
  opacity: 0.6;
  pointer-events: none;
  background-color: #f0f0f0;
}
.task-list {
  list-style-type: none;
  padding-left: 0;
}
</style>
