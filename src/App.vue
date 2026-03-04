<script setup>
    import { reactive } from 'vue';
    import Board from './components/Board.vue'

    export default{
        name: 'App',
        components:{
            Board,
        },
        setup(){
            const boardState = reactive({
                colums:[
                    {id: 'todo', title: 'В работе', maxCards: 3, cards: []},
                    {id: 'in-progress', title: 'На проверке', maxCards: 5, cards: []},
                    {id: 'done', title: 'Готово', maxCards: Infinity, cards: []}
                ],
                nextId: 1
            });

            return{
                boardState,
            }
        }

    }
</script>

<template>
    <div id="app">
        <Board :state="boardState" />
    </div>

</template>

<style scoped>

</style>
