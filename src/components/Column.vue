<!-- Column.vue -->
<template>
  <div class="column">
    <h2>{{ column.title }}</h2>
    <p v-if="column.maxCards !== Infinity">Осталось мест: {{ remainingSlots }}</p>
    <div class="card-list">
      <Card
        v-for="card in column.cards"
        :key="card.id"
        :card="card"
        :state="state"
        :is-locked="isFirstColumn && isLockedForTransfer"
        :move-card="moveCard"
        @progress-changed="checkLockStatus"
      />
    </div>
    <button v-if="canAddCard" @click="addCard">Добавить карточку</button>
    <p v-else-if="column.id === 'todo' && isLockedForTransfer">(Заблокировано)</p>
  </div>
</template>

<script>
import { ref, computed, reactive } from 'vue';
import Card from './Card.vue';

export default {
  name: 'Column',
  components: {
    Card,
  },
  props: {
    column: Object,
    state: Object,
    isFirstColumnFull: Boolean,
    moveCard: Function,
  },
  emits: ['update:isFirstColumnFull'],
  setup(props, { emit }) {
    const isLockedForTransfer = computed(() => {
      if (props.column.id !== 'todo') return false;
      const inProgressCol = props.state.columns.find(c => c.id === 'in-progress');
      return inProgressCol.cards.length >= inProgressCol.maxCards && props.isFirstColumnFull;
    });

    const remainingSlots = computed(() => {
      if (props.column.maxCards === Infinity) return Infinity;
      return props.column.maxCards - props.column.cards.length;
    });

    const canAddCard = computed(() => remainingSlots.value > 0 && !isLockedForTransfer.value);

    const isFirst => props.column.id === 'todo');

    const checkLockStatus = () => {
      if (isFirstColumn.value) {
        let shouldLock = false;
        for (const card of props.column.cards) {
          const progress = card.tasks.filter(t => t.completed).length / card.tasks.length;
          if (progress > 0.5) {
            shouldLock = true;
            break;
          }
        }
        emit('update:isFirstColumnFull', shouldLock);
      }
    };

    const addCard = () => {
      const tasks = Array.from({ length: 3 }, (_, i) => {
        return reactive({
          id: Date.now() + i,
          text: `Задача ${i + 1}`,
          completed: false,
        });
      });

      const newCard = reactive({
        id: props.state.nextId++,
        title: `Карточка ${props.state.nextId - 1}`,
        tasks,
        completedAt: null,
      });

      props.column.cards.push(newCard);
    };

    return {
      isLockedForTransfer,
      remainingSlots,
      canAddCard,
      isFirstColumn,
      checkLockStatus,
      addCard,
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
.card-list {
  min-height: 50px;
  margin-bottom: 10px;
}
</style>