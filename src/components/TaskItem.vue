<template>
  <li>
    <span v-if="!isEditing">{{ task.text }}</span>
    <input
      v-else
      v-model="editText"
      type="text"
      @blur="saveEdit"
      @keyup.enter="saveEdit"
      ref="inputRef"
      :disabled="isDisabled"
    />
    
    <button 
      @click="toggleEdit" 
      style="margin-left: 5px;"
      :disabled="isDisabled" 
    >
      {{ isEditing ? 'Сохранить' : 'Редакт.' }}
    </button>
    <label style="margin-left: 5px;">
      <input
        type="checkbox"
        :checked="task.completed"
        :disabled="isDisabled || isLocked || isEditing"
        @change="toggleTask"
      />
      Выполнено
    </label>
  </li>
</template>

<script>
import { ref, nextTick } from 'vue';

export default {
  name: 'TaskItem',
  props: {
    task: Object,
    isLocked: Boolean,
    isDisabled: Boolean, 
  },
  emits: ['update:completed'],
  setup(props) {
    const isEditing = ref(false);
    const editText = ref('');
    const inputRef = ref(null);

    const toggleEdit = async () => {
      if (props.isDisabled) {
        return;
      }

      if (isEditing.value) {
        saveEdit();
      } else {
        editText.value = props.task.text;
        isEditing.value = true;
        await nextTick();
        if (inputRef.value) {
          inputRef.value.focus();
        }
      }
    };

    const saveEdit = () => {
      if (editText.value.trim()) {
        props.task.text = editText.value.trim();
      }
      isEditing.value = false;
    };

    const toggleTask = (e) => {
      if (!props.isDisabled && !props.isLocked && !isEditing.value) {
        const newValue = e.target.checked;
        props.task.completed = newValue;
        props.$emit('update:completed', { taskId: props.task.id, completed: newValue });
      }
    };

    return {
      isEditing,
      editText,
      inputRef,
      toggleEdit,
      saveEdit,
      toggleTask,
    };
  },
};
</script>