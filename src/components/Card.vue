<template>
  <div class="card" :class="{ locked: isLocked }">
    <!-- Редактируемое название карточки -->
    <div class="card-title">
      <span v-if="!isEditingTitle">{{ card.title }}</span>
      <input
        v-else
        v-model="editTitle"
        type="text"
        @blur="saveTitle"
        @keyup.enter="saveTitle"
        ref="titleInput"
        autofocus
      />
      <button
        @click="toggleTitleEdit"
        style="margin-left: 5px; font-size: 0.8em;"
      >
        {{ isEditingTitle ? 'Сохранить' : 'Ред.' }}
      </button>
    </div>

    <ul class="task-list">
      <TaskItem
        v-for="task in card.tasks"
        :key="task.id"
        :task="task"
        :is-locked="isLocked"
        @update:completed="onTaskCompleted"
      />
    </ul>

    <!-- Форма добавления задачи — только если карточка в первом столбце -->
    <form @submit.prevent="addTask" v-if="isEditable" class="add-task-form">
      <input
        v-model="newTaskText"
        type="text"
        placeholder="Название задачи"
        required
      />
      <button type="submit">Добавить задачу</button>
    </form>

    <p>Прогресс: {{ progress }}%</p>
    <p v-if="card.completedAt">Завершена: {{ card.completedAt }}</p>
  </div>
</template>

<script>
import { computed, ref, nextTick } from 'vue';
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
    isEditable: Boolean, // ← передается из Column.vue
  },
  emits: ['progress-changed'],
  setup(props, { emit }) {
    const editTitle = ref('');
    const isEditingTitle = ref(false);
    const newTaskText = ref('');
    const titleInput = ref(null);

    const progress = computed(() => {
      const total = props.card.tasks.length;
      const done = props.card.tasks.filter(t => t.completed).length;
      return total ? Math.round((done / total) * 100) : 0;
    });

    const onTaskCompleted = () => {
      emit('progress-changed');
    };

    const toggleTitleEdit = async () => {
      if (isEditingTitle.value) {
        saveTitle();
      } else {
        editTitle.value = props.card.title;
        isEditingTitle.value = true;
        await nextTick();
        if (titleInput.value) titleInput.value.focus();
      }
    };

    const saveTitle = () => {
      if (editTitle.value.trim()) {
        props.card.title = editTitle.value.trim();
      }
      isEditingTitle.value = false;
    };

    const addTask = () => {
      if (!newTaskText.value.trim() || !props.isEditable) return;

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
      onTaskCompleted,
      toggleTitleEdit,
      saveTitle,
      addTask,
      isEditingTitle,
      editTitle,
      newTaskText,
      titleInput,
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
.card-title {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}
.card-title input {
  flex: 1;
  padding: 2px 4px;
  border: 1px solid #ccc;
  border-radius: 3px;
}
.task-list {
  list-style-type: none;
  padding-left: 0;
  margin: 8px 0;
}
.add-task-form {
  display: flex;
  gap: 5px;
  margin-top: 5px;
}
.add-task-form input {
  flex: 1;
  padding: 2px 4px;
  border: 1px solid #ccc;
  border-radius: 3px;
}
</style>