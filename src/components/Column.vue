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
        @progress-changed="$emit('progress-updated')"
      />
    </div>
    <form 
      @submit.prevent="addCard" 
      v-if="isFirstColumn && canAddCard" 
      class="add-card-form"
    >
      <input
        v-model="newCardTitle"
        type="text"
        placeholder="Название карточки"
        required
      />
      <button type="submit">Добавить карточку</button>
    </form>
    <p v-else-if="isFirstColumn && isLockedForTransfer">(Заблокировано)</p>
  </div>
</template>

<script>
import { computed, reactive, ref } from 'vue';
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
  },
  emits: ['progress-updated'],
  setup(props) {
    const newCardTitle = ref('');

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

    const addCard = () => {
      if (!newCardTitle.value.trim()) return;

      const tasks = Array.from({ length: 3 }, (_, i) => {
        return reactive({
          id: Date.now() + i,
          text: `Задача ${i + 1}`,
          completed: false,
        });
      });

      const newCard = reactive({
        id: props.state.nextId++,
        title: newCardTitle.value.trim(),
        tasks,
        completedAt: null,
      });

      props.column.cards.push(newCard);
      newCardTitle.value = ''; 
    };

    return {
      isLockedForTransfer,
      remainingSlots,
      canAddCard,
      isFirstColumn,
      addCard,
      newCardTitle,
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
.add-card-form {
  display: flex;
  gap: 5px;
}
.add-card-form input {
  flex: 1;
}
</style>