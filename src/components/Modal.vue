<template>
  <div class="modal-overlay" v-if="visible" @click="close">
    <div class="modal-content" @click.stop>
      <h3>Создать новую карточку</h3>
      <form @submit.prevent="submitForm">
        <div class="form-group">
          <label for="card-title">Название карточки:</label>
          <input
            id="card-title"
            v-model="title"
            type="text"
            required
            :disabled="processing"
          />
        </div>
        
        <div class="form-group">
          <label>Задачи:</label>
          <div 
            v-for="(task, index) in tasks" 
            :key="index" 
            class="task-input-group"
          >
            <input
              v-model="tasks[index]"
              type="text"
              :placeholder="`Задача ${index + 1}`"
              required
            />
            <button 
              type="button" 
              @click="removeTask(index)"
              :disabled="tasks.length <= 3"
            >
              Удалить
            </button>
          </div>
          <button 
            type="button" 
            @click="addTaskField"
            :disabled="tasks.length >= 5"
          >
            Добавить задачу
          </button>
        </div>
        
        <div class="form-actions">
          <button type="submit" :disabled="processing || tasks.length < 3">
            {{ processing ? 'Создание...' : 'Создать' }}
          </button>
          <button type="button" @click="close" :disabled="processing">
            Отмена
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { ref, reactive } from 'vue';

export default {
  name: 'Modal',
  props: {
    visible: {
      type: Boolean,
      required: true
    }
  },
  emits: ['create-card', 'close'],
  setup(props, { emit }) {
    const title = ref('');
    const tasks = reactive(['', '', '']);
    const processing = ref(false);

    const addTaskField = () => {
      if (tasks.length < 5) {
        tasks.push('');
      }
    };

    const removeTask = (index) => {
      if (tasks.length > 3) {
        tasks.splice(index, 1);
      }
    };

    const submitForm = () => {
      if (processing.value) return;
      
      if (!title.value.trim() || tasks.some(t => !t.trim()) || tasks.length < 3) {
        alert('Заполните все поля и добавьте минимум 3 задачи');
        return;
      }

      processing.value = true;
      
      const cardData = {
        title: title.value.trim(),
        tasks: tasks
          .filter(t => t.trim())
          .map(text => ({
            text: text.trim(),
            completed: false
          }))
      };

      emit('create-card', cardData);
      
      setTimeout(() => {
        processing.value = false;
        close();
      }, 300);
    };

    const close = () => {
      if (processing.value) return;
      title.value = '';
      tasks.splice(0);
      tasks.push('', '', '');
      emit('close');
    };

    return {
      title,
      tasks,
      processing,
      addTaskField,
      removeTask,
      submitForm,
      close
    };
  }
};
</script>

<style scoped>

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  min-width: 400px;
  max-width: 500px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.task-input-group {
  display: flex;
  gap: 5px;
  margin-bottom: 5px;
}

.task-input-group input {
  flex: 1;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 20px;
}
</style>