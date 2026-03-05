<!-- Card.vue -->
<template>
  <div class="card" :class="{ locked: isLocked }">
    <h3>{{ card.title }}</h3>
    <ul class="task-list">
      <TaskItem
        v-for="task in card.tasks"
        :key="task.id"
        :task="task"
        :is-locked="isLocked"
        @update:completed="onTaskCompleted"
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

    const onTaskCompleted = () => {
      emit('progress-changed');
    };

    return {
      progress,
      onTaskCompleted,
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