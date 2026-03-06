<template>
  <div class="card" :class="{ locked: isLocked }">
    <h3>{{ card.title }}</h3>
    <ul class="task-list">
      <TaskItem
        v-for="task in card.tasks"
        :key="task.id"
        :task="task"
        :is-locked="isLocked"
        :is-disabled="isInDoneColumn"
        @update:completed="onTaskCompleted"
      />
    </ul>
    <form v-if="isInFirstColumn && canAddTask" @submit.prevent="addTask" class="add-task-form">
      <input
        v-model="newTaskText"
        type="text"
        placeholder="Название задачи"
        required
      />
      <button type="submit">Добавить задачу</button>
    </form>
    <p v-if="isInFirstColumn && !canAddTask" class="limit-message">
      Достигнут лимит задач (максимум 5)
    </p>
    <p>Прогресс: {{ progress }}%</p>
    <p v-if="card.completedAt">Завершена: {{ card.completedAt }}</p>
  </div>
</template>

<script>
import { computed, ref } from 'vue';
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
    isInFirstColumn: Boolean,
    isInDoneColumn: Boolean, // Новый пропс
  },
  emits: ['progress-changed'],
  setup(props, { emit }) {
    const newTaskText = ref('');

    const progress = computed(() => {
      const total = props.card.tasks.length;
      const done = props.card.tasks.filter(t => t.completed).length;
      return total ? Math.round((done / total) * 100) : 0;
    });

    const canAddTask = computed(() => {
      return props.isInFirstColumn && props.card.tasks.length < 5;
    });

    const onTaskCompleted = () => {
      emit('progress-changed');
    };

    const addTask = () => {
      if (props.card.tasks.length >= 5) {
        console.warn("Нельзя добавить больше 5 задач");
        return;
      }

      if (!newTaskText.value.trim()) return;

      const newTask = {
        id: Date.now(),
        text: newTaskText.value.trim(),
        completed: false,
      };

      props.card.tasks.push(newTask);
      newTaskText.value = '';
    };

    return {
      progress,
      canAddTask,
      onTaskCompleted,
      addTask,
      newTaskText,
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
  background-color: #f0f0f0;
}
.task-list {
  list-style-type: none;
  padding-left: 0;
}
.add-task-form {
  display: flex;
  gap: 5px;
  margin-top: 5px;
}
.add-task-form input {
  flex: 1;
}
.limit-message {
  color: #666;
  font-size: 0.9em;
  margin-top: 5px;
  margin-bottom: 0;
}
</style>