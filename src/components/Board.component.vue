<script setup>
    import { ref, computed, watch, onMounted, onBeforeUnmount  } from 'vue'
    import { v4 as uuidv4 } from 'uuid'


    const { 
        onBoardChange, 
        FEN, 
        flip, 
        moveCopyPiece, 
        setMoveCopyPiece,
        isErasing,
        handleErase,
        stockfishResponse,
        } = 
    defineProps({ 
        onBoardChange: Function, 
        FEN: String,
        flip: Boolean,
        moveCopyPiece: String,
        setMoveCopyPiece: Function,
        isErasing: Boolean,
        handleErase: Function,
        stockfishResponse: Object
    })

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
        {A: 'r', B: 'n', C: 'b', D: 'q', E: 'k', F: 'b', G: 'n',H: 'r'},
        {A: 'p', B: 'p', C: 'p', D: 'p', E: 'p', F: 'p', G: 'p',H: 'p'},
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: '',  B: '',  C: '',  D: '',  E: '',  F: '',  G: '', H: '' },
        {A: 'P', B: 'P', C: 'P', D: 'P', E: 'P', F: 'P', G: 'P',H: 'P'},
        {A: 'R', B: 'N', C: 'B', D: 'Q', E: 'K', F: 'B', G: 'N',H: 'R'}
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
    let updating = false

    //---------------------------FUNCTIONS

    const isEven = (number) => {
        return number % 2 == 0
    }

    const isBest = (row, col) => {
        if(currentFlip) {
            
        }
        return (row == bestMoveFromRow.value && col == bestMoveFromColumn.value) || (row == bestMoveToRow.value && col == bestMoveToColumn.value) 
    }

    const isSquareClicked = (row, col) => {

        return (
            (moveFrom &&
            moveFrom.row &&
            moveFrom.col &&
            moveFrom.row === row &&
            moveFrom.col === col) ||
            (dragFrom &&
            dragFrom.row &&
            dragFrom.col &&
            dragFrom.row === row &&
            dragFrom.col === col)
        );
    }

    const onDragStart = (row, col, event) => {

        dragFrom = { row, col }
        event.target.style.opacity = 0

        setMoveCopyPiece(board.value[row][col])

        event.dataTransfer.effectAllowed = "move"
        event.dataTransfer.dropEffect = "move"

    }

    const onDrop = (toRow, toCol, event) => {
        if (!dragFrom && !currentMoveCopyPiece.value) return

        if(dragFrom) board.value[dragFrom.row][dragFrom.col] = ''
        if(currentMoveCopyPiece.value) board.value[toRow][toCol] = currentMoveCopyPiece.value

        setMoveCopyPiece(null)
        dragFrom = null
    }

    const onDragEnd = (event) => {
        event.target.style.opacity = 1
    }

    const moveByClick = (row, col, event) => {
        if(moveFrom) {
            board.value[moveFrom.row][moveFrom.col] = ''
            board.value[row][col] = currentMoveCopyPiece.value
            
            setMoveCopyPiece(null)
            moveFrom = null
        }
        else if (currentMoveCopyPiece.value || currentIsErasing.value) {
            board.value[row][col] = currentIsErasing.value ? '' : currentMoveCopyPiece.value
            setMoveCopyPiece(null)
        }
        else {
            setMoveCopyPiece(board.value[row][col])
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
            if(index != 7) temp_fen += '/'
        })
        return temp_fen
    }

    //----------------------------REFS

    const board = ref(initial_board)
    const bestMoveFromRow = ref('')
    const bestMoveFromColumn = ref('')
    const bestMoveToRow = ref('')
    const bestMoveToColumn = ref('')

    //----------------------------COMPUTED

    const currentFEN = computed(() => FEN)
    const currentFlip = computed(() => flip)
    const currentMoveCopyPiece = computed(() => moveCopyPiece)
    const currentIsErasing = computed(() => isErasing)
    const currentStockfishResponse = computed(() => stockfishResponse)

    //----------------------------WATCH

    watch(board, (newVal, oldVal) => {
        if(updating) return

        updating = true
        onBoardChange(getFEN())
        setTimeout(() => {
            updating = false
        }, 500)
    }, { deep: true })

    watch(currentFEN, (newVal, oldVal) => {
        if(updating) return

        updating = true
        var position = -1
        const new_board = []
        newVal.split('/').forEach((row, row_index) => {
            const board_row = {}
            row.split('').forEach(square => {
                position++
                if(isNaN(square)) board_row[letters[position]] = square
                else {
                    for(var i = position; i < position + square; i++) {
                        board_row[letters[i]] = ''
                    }
                    position += square - 1
                }
            })
            position = -1
            new_board.push(board_row)
        })
        board.value = new_board
        setTimeout(() => {
            updating = false
        }, 500)
    })

    watch(currentStockfishResponse, (newVal, oldVal) => {
        if(newVal.success) {
            if(newVal.bestmove) {
                const bestmove = newVal.bestmove.split(' ')[1].split('')
                bestMoveFromRow.value = Math.abs(parseInt(bestmove[1]) - 8)
                bestMoveFromColumn.value = bestmove[0].toUpperCase()
                bestMoveToRow.value = Math.abs(parseInt(bestmove[3]) - 8)
                bestMoveToColumn.value = bestmove[2].toUpperCase()
            }
        }
    })
    //-----------------------------INIT

    onBoardChange(getFEN())

</script>

<template>
    <div 
        class="board__container"
        :class="[currentFlip ? 'rotate' : '']"
        @contextmenu.prevent
    >
        <div v-for="(number, number_index) in numbers_indexed" :key="number.id" class="board__container_row">
            <div
                v-for="(letter, letter_index) in letters_indexed"
                :key="letter.id" class="board__container_row_square"
                :class="[isEven(letter_index + number_index + 2) ? 'light' : 'dark' 
                    ,isSquareClicked(number_index, letter.value) ? 'selected' : ''
                    ,isBest(number_index, letter.value) ? 'best' : '']"
                @dragover.prevent
                @drop="(e) => onDrop(number_index, letter.value, e)"
                @click="(e) => moveByClick(number_index, letter.value, e)"
            >
                <img 
                    v-if="board[number_index][letter.value] != ''"
                    :src="`chess-pieces/${dictionary[board[number_index][letter.value]]}.png`"
                    class="board__container_row_square-piece-image"
                    :class="[currentFlip ? 'rotate' : '']"
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
                width: min(99svh / 8, 100svw / 8);
                height: min(99svh / 8, 100svw / 8);
                flex: 1;
                aspect-ratio: 1/1;

                .board__container_row_square-piece-image {

                    height: 100%;
                    width: 100%;
                    object-fit: contain;

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

    .selected {
        background-color: #d9ff00b0;
    }

    .best {
        background-color: #00ddffb0;
    }

</style>
