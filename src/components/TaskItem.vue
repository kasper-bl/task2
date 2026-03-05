<!-- TaskItem.vue -->
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
        this.task.completed = e.target.checked;
      }
    },
  },
};
</script>