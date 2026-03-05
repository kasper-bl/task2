<template>
  <li>
    <label>
      <input
        type="checkbox"
        :checked="task.completed"
        :disabled="isLocked"
        @change="toggleTask"
      />
      {{ task.text }}
    </label>
  </li>
</template>

<script>
export default {
  name: 'TaskItem',
  props: {
    task: Object,
    isLocked: Boolean,
  },
  methods: {
    toggleTask(e) {
      if (!this.isLocked) {
        const newValue = e.target.checked;
        this.task.completed = newValue;
        this.$emit('update:completed', { taskId: this.task.id, completed: newValue });
      }
    },
  },
};
</script>