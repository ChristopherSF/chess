<script setup>
    import { ref } from 'vue'
    import Board from '../components/Board.component.vue'
    import Controls from '../components/Controls.component.vue'

    const FEN = ref('')

    const onBoardChange = (newFEN) => {
        FEN.value = newFEN
    }
</script>

<template>
    <main>
        <Board :onBoardChange="onBoardChange"/>
        <Controls />
    </main>
</template>

<style scoped>
    main {

        display: flex;
        align-items: center;
        justify-content: space-around;

        flex-wrap: wrap;

        width: 100%;
        min-height: 95svh;

    }
</style>
