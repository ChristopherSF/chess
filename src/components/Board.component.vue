<script setup>
    import { ref, computed, watch, onMounted, onBeforeUnmount  } from 'vue'
    import { v4 as uuidv4 } from 'uuid'


    const { onBoardChange, FEN } = defineProps({ onBoardChange: Function, FEN: String })

    //-----------------------------CONSTS

    const letters = ['A','B','C','D','E','F','G','H']
    const numbers = [1,2,3,4,5,6,7,8]
    const dictionary = {
        P: 'pawn_w',
        N: 'knight_w',
        B: 'bishop_w',
        R: 'rook_w',
        Q: 'queen_w',
        K: 'king_w',
        p: 'pawn_b',
        n: 'knight_b',
        b: 'bishop_b',
        r: 'rook_b',
        q: 'queen_b',
        k: 'king_b',
    }

    const initial_board = [
        {A: 'R', B: 'N', C: 'B', D: 'Q', E: 'K', F: 'B', G: 'N',H: 'R'},
        {A: 'P', B: 'P', C: 'P', D: 'P', E: 'P', F: 'P', G: 'P',H: 'P'},
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: 'p', B: 'p', C: 'p', D: 'p', E: 'p', F: 'p', G: 'p',H: 'p'},
        {A: 'r', B: 'n', C: 'b', D: 'q', E: 'k', F: 'b', G: 'n',H: 'r'}
    ]

    const letters_indexed = letters.map(letter => ({
        id: uuidv4(),
        value: letter
    }));

    const numbers_indexed = numbers.map(number => ({
        id: uuidv4(),
        value: number
    }))

    let dragFrom = null
    let moveFrom = null

    //---------------------------FUNCTIONS

    const isEven = (number) => {
        return number % 2 == 0
    }

    const onDragStart = (row, col, event) => {

        dragFrom = { row, col }
        event.target.style.opacity = 0

        event.dataTransfer.effectAllowed = "move"
        event.dataTransfer.dropEffect = "move"

    }

    const onDrop = (toRow, toCol, event) => {
        if (!dragFrom) return

        const piece = board.value[dragFrom.row][dragFrom.col]
        board.value[dragFrom.row][dragFrom.col] = ''
        board.value[toRow][toCol] = piece

        dragFrom = null
    }

    const onDragEnd = (event) => {
        event.target.style.opacity = 1
    }

    /*const onTouchStart = (row, col, event) => {
        dragFrom = { row, col }
        event.target.style.opacity = 0
    }

    const onTouchEnd = (toRow, toCol, event) => {
        if (!dragFrom) return

        const piece = board.value[dragFrom.row][dragFrom.col]
        board.value[dragFrom.row][dragFrom.col] = ''
        board.value[toRow][toCol] = piece

        event.target.style.opacity = 1
    }*/

    const moveByClick = (row, col, event) => {
        if(moveFrom) {
            const piece = board.value[moveFrom.row][moveFrom.col]
            board.value[moveFrom.row][moveFrom.col] = ''
            board.value[row][col] = piece
            moveFrom = null
        }
        else {
            moveFrom = { row, col }
        }
    }

    const getFEN = () => {
        var temp_counter = 0
        var temp_fen = ''
        board.value.forEach((row, index) => {
            Object.values(row).forEach((square) => {
                if (square != '') {
                    if (temp_counter > 0) { 
                        temp_fen += temp_counter
                        temp_counter = 0
                    }
                    temp_fen += square
                }
                else temp_counter++
            })
            if (temp_counter > 0) { 
                temp_fen += temp_counter
                temp_counter = 0
            }
            if(index != 7)   temp_fen += '/'
        })
        return temp_fen
    }

    //----------------------------REFS

    const board = ref(initial_board)
    const touchedPiece = ref(null)
    const dragging = ref(false)
    const position = ref({ x: 0, y: 0 })

    //----------------------------COMPUTED

    const currentFEN = computed(() => FEN)

    //----------------------------WATCH

    watch(board, (newVal, oldVal) => {
        onBoardChange(getFEN())
    }, { deep: true })

    watch(currentFEN, (newVal, oldVal) => {
        console.log(newVal)
    })

    //----------------------------

</script>

<template>
    <div 
        class="board__container"
        @contextmenu.prevent
    >
        <div v-for="(number, number_index) in numbers_indexed" :key="number.id" class="board__container_row">
            <div
                v-for="(letter, letter_index) in letters_indexed"
                :key="letter.id" class="board__container_row_square"
                :class="[isEven(letter_index + number_index + 2) ? 'light' : 'dark']"
                @dragover.prevent
                @drop="(e) => onDrop(number_index, letter.value, e)"
                @click="(e) => moveByClick(number_index, letter.value, e)"
            >
                <img 
                    v-if="board[number_index][letter.value] != ''"
                    :src="`/chess-pieces/${dictionary[board[number_index][letter.value]]}.png`"
                    class="board__container_row_square-piece-image"
                    draggable="true"
                    @dragstart="(e) => onDragStart(number_index, letter.value, e)"
                    @dragend="(e) => onDragEnd(e)"
                >
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>

    .board__container {

        max-width: 100svw;
        max-height: 99svh;
        aspect-ratio: 1 / 1;

        .board__container_row {

            display: flex;
            flex-direction: row;

            .board__container_row_square {
                min-width: 30px;
                min-height: 30px;
                flex: 1;
                aspect-ratio: 1/1;

                .board__container_row_square-piece-image {

                    height: 100%;
                    width: 100%;

                    &:hover {
                        cursor: grab;
                    }

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

    .rotate {
        transform: rotate(180deg);
    }

</style>
