<template>
  <div class="column">
    <div class="column-header">
      <h2>{{ column.title }}</h2>
      <button 
        v-if="isFirstColumn && canAddCard" 
        @click="openModal"
        :disabled="isLockedForTransfer"
        class="add-card-btn"
      >
        +
      </button>
    </div>
    
    <p v-if="column.maxCards !== Infinity">Осталось мест: {{ remainingSlots }}</p>
    
    <div class="card-list">
      <Card
        v-for="card in column.cards"
        :key="card.id"
        :card="card"
        :state="state"
        :is-locked="isFirstColumn && isLockedForTransfer"
        :is-in-first-column="isFirstColumn"
        :is-in-done-column="isDoneColumn"
        @progress-changed="$emit('progress-updated')"
      />
    </div>
    
    <p v-if="isFirstColumn && isLockedForTransfer" class="lock-message">
      (Заблокировано)
    </p>
    
    <Modal 
      :visible="showModal" 
      @create-card="handleCreateCard" 
      @close="closeModal" 
    />
  </div>
</template>

<script>
import { computed, ref } from 'vue';
import Card from './Card.vue';
import Modal from './Modal.vue';

export default {
  name: 'Column',
  components: {
    Card,
    Modal,
  },
  props: {
    column: Object,
    state: Object,
    isFirstColumnFull: Boolean,
  },
  emits: ['progress-updated'],
  setup(props) {
    const showModal = ref(false);

    const isLockedForTransfer = computed(() => {
      if (props.column.id !== 'todo') return false;
      const inProgressCol = props.state.columns.find(c => c.id === 'in-progress');
      return inProgressCol && inProgressCol.cards.length >= inProgressCol.maxCards && props.isFirstColumnFull;
    });

    const remainingSlots = computed(() => {
      if (props.column.maxCards === Infinity) return Infinity;
      return props.column.maxCards - props.column.cards.length;
    });

    const canAddCard = computed(() => {
      return props.column.id === 'todo' && remainingSlots.value > 0 && !isLockedForTransfer.value;
    });

    const isFirstColumn = computed(() => props.column.id === 'todo');
    const isDoneColumn = computed(() => props.column.id === 'done');

    const openModal = () => {
      if (canAddCard.value && !isLockedForTransfer.value) {
        showModal.value = true;
      }
    };

    const closeModal = () => {
      showModal.value = false;
    };

    const handleCreateCard = (cardData) => {
      if (remainingSlots.value <= 0) {
        alert('Достигнут лимит карточек в этом столбце');
        return;
      }

      const newCard = {
        id: props.state.nextId++,
        title: cardData.title,
        tasks: cardData.tasks.map((task, index) => ({
          id: Date.now() + index,
          text: task.text,
          completed: task.completed
        })),
        completedAt: null
      };

      props.column.cards.push(newCard);
      closeModal();
    };

    return {
      isLockedForTransfer,
      remainingSlots,
      canAddCard,
      isFirstColumn,
      isDoneColumn,
      showModal,
      openModal,
      closeModal,
      handleCreateCard,
    };
  },
};
</script>

<style scoped>
.column {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 10px;
  width: 300px;
  background-color: #f9f9f9;
}

.column-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.add-card-btn {
  width: 30px;
  height: 30px;
  border: none;
  background-color: #535353;
  color: white;
  border-radius: 20px;
  cursor: pointer;
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-card-btn:hover:not(:disabled) {
  background-color: #6d6d6d;
}

.add-card-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.card-list {
  min-height: 50px;
  margin-bottom: 10px;
}

.lock-message {
  color: #666;
  font-size: 0.9em;
  margin-top: 5px;
  margin-bottom: 0;
}
</style>