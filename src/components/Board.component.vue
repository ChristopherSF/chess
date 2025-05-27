<script setup>
    import { ref } from 'vue'

    defineProps({
        msg: String,
    })

    const count = ref(0)

    var letters = ['A','B','C','D','E','F','G','H']
    var numbers = [1,2,3,4,5,6,7,8]

    const initial_board = [
        {A: 'rook_b', B: 'knight_b', C: 'bishop_b', D: 'queen_b', E: 'king_b', F: 'bishop_b', G: 'knight_b',H: 'rook_b'},
        {A: 'pawn_b', B: 'pawn_b', C: 'pawn_b', D: 'pawn_b', E: 'pawn_b', F: 'pawn_b', G: 'pawn_b',H: 'pawn_b'},
        {A: '', B: '', C: '', D: '', E: '', F: '', G: '',H: ''},
        {A: '', B: '', C: '', D: '', E: '', F: '', G: '',H: ''},
        {A: '', B: '', C: '', D: '', E: '', F: '', G: '',H: ''},
        {A: '', B: '', C: '', D: '', E: '', F: '', G: '',H: ''},
        {A: 'pawn_w', B: 'pawn_w', C: 'pawn_w', D: 'pawn_w', E: 'pawn_w', F: 'pawn_w', G: 'pawn_w',H: 'pawn_w'},
        {A: 'rook_w', B: 'knight_w', C: 'bishop_w', D: 'queen_w', E: 'king_w', F: 'bishop_w', G: 'knight_w',H: 'rook_w'},
    ]

    var board = initial_board

    const letters_indexed = letters.map(letter => ({
        id: crypto.randomUUID(),
        value: letter
    }));

    const numbers_indexed = numbers.map(number => ({
        id: crypto.randomUUID(),
        value: number
    }))

    const isEven = (number) => {
        return number % 2 == 0
    }

</script>

<template>
    <div class="board__container">
        <div v-for="(number, n_i) in numbers_indexed" :key="number.id" class="board__container_row">
            <div v-for="(letter, l_i) in letters_indexed" :key="letter.id" :class="['board__container_row_square', isEven(l_i + n_i + 2) ? 'light' : 'dark']">
                <img v-if="board[n_i][letter.value] != ''" :src="`src/assets/chess-pieces-background/${board[n_i][letter.value]}.png`" class="board__container_row_square-piece-image">
            </div>
        </div>
    </div>

</template>

<style lang="scss" scoped>

    .board__container {

        .board__container_row {

            display: flex;
            flex-direction: row;

            .board__container_row_square {
                height: 100px;
                width: 100px;

                .board__container_row_square-piece-image {

                    height: 100px;
                    width: 100px;

                }

            }

        }

    }

    .dark {
        background-color: #749454;
    }

    .light {
        background-color: #eaedd0;
    }

</style>
