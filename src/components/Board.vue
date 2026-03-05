<!-- Board.vue -->
<template>
  <div class="board">
    <h1>Todo</h1>
    <div class="columns">
      <Column
        v-for="col in state.columns"
        :key="col.id"
        :column="col"
        :state="state"
        :is-first-column-full="isFirstColumnFull"
        :move-card="moveCard"
      />
    </div>
  </div>
</template>

<script>
import { ref, watchEffect, reactive } from 'vue';
import Column from './Column.vue';

export default {
  name: 'Board',
  components: {
    Column,
  },
  props: {
    state: Object,
  },
  setup(props) {
    // Загрузка данных из localStorage
    const saved = localStorage.getItem('boardState');
    if (saved) {
      const parsed = JSON.parse(saved);
      props.state.columns[0].cards = parsed.columns[0].cards || [];
      props.state.columns[1].cards = parsed.columns[1].cards || [];
      props.state.columns[2].cards = parsed.columns[2].cards || [];
      props.state.nextId = parsed.nextId || 1;
    }

    // Сохранение данных в localStorage
    watchEffect(() => {
      const toSave = {
        columns: props.state.columns.map(col => ({
          id: col.id,
          cards: col.cards.map(c => ({ ...c })),
        })),
        nextId: props.state.nextId,
      };
      localStorage.setItem('boardState', JSON.stringify(toSave));
    });

    const isFirstColumnFull = ref(false);

    // Функция перемещения карточки между колонками
    const moveCard = (card, progressValue) => {
      // Находим текущую колонку карточки
      let currentColIndex = -1;
      let cardIndexInCol = -1;

      for (let i = 0; i < props.state.columns.length; i++) {
        const index = props.state.columns[i].cards.findIndex(c => c.id === card.id);
        if (index !== -1) {
          currentColIndex = i;
          cardIndexInCol = index;
          break;
        }
      }

      // Если карточка не найдена, выходим
      if (currentColIndex === -1 || cardIndexInCol === -1) {
        console.log("Карточка не найдена в текущих колонках");
        return;
      }

      // Получаем текущую колонку
      const currentColumn = props.state.columns[currentColIndex];

      console.log(`Найдена карточка в колонке ${currentColumn.id}, прогресс: ${progressValue}%`);

      // Условия для перемещения
      if (currentColIndex === 0 && progressValue > 50) {
        // Перемещаем из todo в in-progress
        const cardToMove = currentColumn.cards.splice(cardIndexInCol, 1)[0];
        const inProgressCol = props.state.columns.find(c => c.id === 'in-progress');
        if (inProgressCol && inProgressCol.cards.length < inProgressCol.maxCards) {
          inProgressCol.cards.push(cardToMove);
          console.log("Карточка перемещена из 'В работе' в 'На проверке'");
        } else {
          console.log("Нельзя переместить - колонка 'На проверке' заполнена");
          // Возвращаем карточку обратно, если перемещение невозможно
          currentColumn.cards.splice(cardIndexInCol, 0, cardToMove);
        }
      } 
      else if (currentColIndex === 1 && progressValue === 100) {
        // Перемещаем из in-progress в done
        const cardToMove = currentColumn.cards.splice(cardIndexInCol, 1)[0];
        cardToMove.completedAt = new Date().toLocaleString();
        const doneCol = props.state.columns.find(c => c.id === 'done');
        if (doneCol) {
          doneCol.cards.push(cardToMove);
          console.log("Карточка перемещена из 'На проверке' в 'Готово'");
        } else {
          // Возвращаем карточку обратно, если перемещение невозможно
          currentColumn.cards.splice(cardIndexInCol, 0, cardToMove);
        }
      }
    };

    return {
      isFirstColumnFull,
      moveCard,
    };
  },
};
</script>

<style scoped>
.board {
  padding: 20px;
}
.columns {
  display: flex;
  gap: 20px;
}
</style>