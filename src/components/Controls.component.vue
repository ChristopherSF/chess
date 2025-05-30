<script setup>

    import { ref, computed, watch } from 'vue' 
    import { v4 as uuidv4 } from 'uuid'

    const { 
        onBoardChange, 
        FEN, 
        flip, 
        moveCopyPiece, 
        setMoveCopyPiece, 
        handleFlip,
        isErasing,
        handleErase,
        setCastleWhiteKing,
        setCastleWhiteQueen,
        setCastleBlackKing,
        setCastleBlackQueen,
        setPassant,
        setColorMoves,
        stockfishResponse
        } = 
    defineProps({ 
        onBoardChange: Function, 
        FEN: String,
        flip: Boolean,
        moveCopyPiece: String,
        setMoveCopyPiece: Function,
        handleFlip: Function,
        isErasing: Boolean,
        handleErase: Function,
        setCastleWhiteKing: Function,
        setCastleWhiteQueen: Function,
        setCastleBlackKing: Function,
        setCastleBlackQueen: Function,
        setPassant: Function,
        setColorMoves: Function,
        stockfishResponse: Object
    })

    const pieces = [
        'p',
        'n',
        'b',
        'r',
        'q',
        'k'
    ]

    const pieces_indexed = pieces.map(piece => ({
        value: piece,
        id: uuidv4()
    }));

    const empty_board = '8/8/8/8/8/8/8/8'
    const initial_board = 'rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR'
    const dictionary = {
        p: 'pawn',
        n: 'knight',
        b: 'bishop',
        r: 'rook',
        q: 'queen',
        k: 'king',
    }
    const dictionary_images = {
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
    const letters = ['a','b','c','d','e','f','g','h']

    //------------------------------------COMPUTED

    const currentMoveCopyPiece = computed(() => moveCopyPiece)
    const currentFlip = computed(() => flip)
    const currentIsErasing = computed(() => isErasing)
    const currentStockfishReponse = computed(() => stockfishResponse)

    //------------------------------------FORM INFO

    const currentColorMoves = ref('w')
    const currentCastleWhiteKing = ref(false)
    const currentCastleWhiteQueen = ref(false)
    const currentCastleBlackKing = ref(false)
    const currentCastleBlackQueen = ref(false)
    const currentPassant = ref('-')
    const currentFEN = ref(FEN)

    const handleClickColorMoves = () => {
        const value = currentColorMoves.value == 'w' ? 'b' : 'w'
        currentColorMoves.value = value
        setColorMoves(value)
    }
    const handleChangeCastleWhiteKing = (event) => {
        setCastleWhiteKing(currentCastleWhiteKing ? 'K' : '')
    }
    const handleChangeCastleWhiteQueen = (event) => {
        setCastleWhiteQueen(currentCastleWhiteQueen ? 'Q' : '')
    }
    const handleChangeCastleBlackKing = (event) => {
        setCastleBlackKing(currentCastleBlackKing ? 'k' : '')
    }
    const handleChangeCastleBlackQueen = (event) => {
        setCastleBlackQueen(currentCastleBlackQueen ? 'q' : '')
    }
    const handleChangePassant = (event) => {
        setPassant(currentPassant)
    }
    const handleChangeFEN = (event) => {
        if(event.key == 'Enter') onBoardChange(currentFEN.value)
    }

    //----------------------------------FUNCTIONS

    const convertColorPiece = (piece) => {
        return !colorPieces.value ? piece : piece.toUpperCase()
    }

   //-----------------------------------ACTIONS

    const colorPieces = ref(true)
    const orientation = ref(true)
    const percentage = ref(0)
    const barPercentage = ref(50)

    const handleClickColorPieces = () => colorPieces.value = !colorPieces.value
    const handleClickClearBoard = () => {
        currentFEN.value = empty_board
        onBoardChange(currentFEN.value)
    }
    const handleClickInitialBoard = () => {
        currentFEN.value = initial_board
        onBoardChange(currentFEN.value)
    }

    const handleDragStartOnCopyImage = (piece, event) => {
        setMoveCopyPiece(convertColorPiece(piece))
        event.dataTransfer.effectAllowed = "move"
        event.dataTransfer.dropEffect = "move"
    }

    const handleClickOnCopyImage = (piece, event) => {
        if(currentMoveCopyPiece.value == convertColorPiece(piece)) setMoveCopyPiece('')
        else setMoveCopyPiece(convertColorPiece(piece))
    }

    const handleClickFlip = () => {
        handleFlip()
    }

    const handleClickErase = () => {
        handleErase()
    }

    //------------------------------------WATCH
    
    watch(() => FEN, (newVal, oldVal) => {
        currentFEN.value = newVal
    })

    watch(currentStockfishReponse, (newVal, oldVal) => {
        if(newVal.success) {
            if(newVal.evaluation) {
                const evaluation_white = 50 + (parseFloat(newVal.evaluation) * 2)
                const percentage = evaluation_white > 95 ? 95 : evaluation_white 
                barPercentage.value = percentage
            }
            if(newVal.mate) {
                barPercentage.value = currentColorMoves.value == 'w' ? 100 : 0
            }
        }
    })

</script>

<template>
    <aside>
        <label class="aside__container-title">Stockfish Analyzer</label>
        <div class="aside__container_analize">
            <label v-if="currentStockfishReponse.evaluation" class="aside__container_analize-percentage">{{ `${(parseFloat(currentStockfishReponse.evaluation) > 0) ? '+' : ''}${currentStockfishReponse.evaluation}` }}</label>
            <label v-if="currentStockfishReponse.mate" class="aside__container_analize-percentage">{{ `#${currentStockfishReponse.mate}` }}</label>
            <div class="aside__container_analize_bar">
                <div class="aside__container_analize_bar-increment" :style="{width: barPercentage + '%'}"></div>
            </div>
        </div>
        <div class="aside__container_pieces">
            <div
                class="aside__container_pieces-color"
                :class="[colorPieces ? 'white' : 'black']"
                @click="handleClickColorPieces"
            ></div>
            <div class="aside__container_pieces_container">
                <img 
                    v-for="p in pieces_indexed"
                    :key="p.id" :src="`chess-pieces/${dictionary[p.value]}${colorPieces ? '_w' : '_b'}.png`"
                    class="aside__container_pieces_container-item"
                    :class="[convertColorPiece(p.value) == currentMoveCopyPiece ? 'selected' : '']"
                    @dragstart="(e) => handleDragStartOnCopyImage(p.value, e)"
                    @click="() => handleClickOnCopyImage(p.value)"
                >
            </div>
        </div>
        <div class="aside__container_editors">
            <button 
                class="aside__container_editors-flip" 
                :class="[currentFlip ? 'selected' : '']"
                @click="handleClickFlip"
            >
                Flip
            </button>
            <button class="aside__container_editors-clear-board" @click="handleClickClearBoard">Clear Board</button>
            <button class="aside__container_editors-initial-board" @click="handleClickInitialBoard">Initial Board</button>
            <button 
                class="aside__container_editors-eraser"
                :class="[currentIsErasing ? 'selected' : '']"
                @click="handleClickErase"
            >
                Eraser
            </button>
        </div>
        <div class="aside__container_fen">
            <label class="aside__container_fen-head">FEN</label>
            <input class="aside__container_fen-input" type="text" v-model="currentFEN" @keydown="(e) => handleChangeFEN(e)">
        </div>
        <div class="aside__container_moves">
            <label class="aside__container_moves-head">Moves</label>
            <div
                class="aside__container_moves-color"
                :class="[currentColorMoves == 'w' ? 'white' : 'black']"
                @click="handleClickColorMoves"
            ></div>
        </div>
        <div class="aside__container_castle">
            <label class="aside__container_castle-head">Castle</label>
            <div class="aside__container_castle_white">
                <div class="aside__container_castle_white_circle"></div>
                <input v-model="currentCastleWhiteKing" @change="handleChangeCastleWhiteKing" class="aside__container_castle_white-input" type="checkbox" id="white">
                <label class="aside__container_castle_white-head" for="white">King</label>
                <input v-model="currentCastleWhiteQueen" @change="handleChangeCastleWhiteQueen" class="aside__container_castle_white-input" type="checkbox" id="white">
                <label class="aside__container_castle_white-head" for="white">Queen</label>
            </div>
            <div class="aside__container_castle_black">
                <div class="aside__container_castle_black_circle"></div>
                <input v-model="currentCastleBlackKing" @change="handleChangeCastleBlackKing" class="aside__container_castle_black-input" type="checkbox" id="black">
                <label class="aside__container_castle_black-head" for="black">King</label>
                <input v-model="currentCastleBlackQueen" @change="handleChangeCastleBlackQueen" class="aside__container_castle_black-input" type="checkbox" id="black">
                <label class="aside__container_castle_black-head" for="black">Queen</label>
            </div>
        </div>
        <div class="aside__container_passant">
            <label class="aside__container_passant-head" for="passant">Passant</label>   
            <input v-model="currentPassant" @change="handleChangePassant" class="aside__container_passant-input" type="text" id="passant">     
        </div>
    </aside>
</template>

<style lang="scss" scoped>
    aside {

        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        gap: 20px;
        padding: 20px;
        padding-top: 40px;
        padding-bottom: 20px;
        width: clamp(300px, 50%, 600px);

        .aside__container_next_movements {
            display: flex;
            gap: 10px;
        }

        .aside__container-title {
            text-align: center;
            font-size: 2rem;
            font-weight: 400;
            width: 100%;
        }

        .aside__container_fen {

            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            .aside__container_fen-head {
                background: var(--second-background);
                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;
            }

            .aside__container_fen-input {
                font-size: 1rem;
                height: 35px;
                border: none;
                text-align: center;
                width: clamp(100px, 70%, 500px);
            }

        }

        .aside__container_analize {
            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            .aside__container_analize-percentage {
                background: var(--second-background);
                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;
            }

            .aside__container_analize_bar {
                background: black;
                height: 35px;
                width: clamp(100px, 70%, 500px);

                .aside__container_analize_bar-increment {
                    background: white;
                    height: 35px;
                    width: 50%;
                }
            }
        }

        .aside__container_pieces {
            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            .aside__container_pieces-color {
                width: 30px;
                height: 30px;
                border-radius: 50%;
                border: 1px solid white;

                cursor: pointer;
            }

            .aside__container_pieces_container {

                display: flex;
                align-items: center;
                justify-content: center;

                .aside__container_pieces_container-item {
                    width: 40px;
                    height: 40px;

                    cursor: pointer;
                }

            }

        }

        .aside__container_editors {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 90%;
            gap: 10px;

            button {
                font-size: 1.2rem;
                background: var(--second-background);
                border: none;
                padding: 10px;

                cursor: pointer;
            }
        }

        .aside__container_moves {

            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            .aside__container_moves-head {

                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;

            }
            .aside__container_moves-color {
                width: 30px;
                height: 30px;
                border-radius: 50%;
                border: 1px solid white;

                cursor: pointer;
            }

        }

        .aside__container_castle {
            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-direction: column;
            gap: 20px;

            .aside__container_castle-head {

                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;

            }

            .aside__container_castle_white {

                display: flex;
                align-items: center;
                justify-content: center;
                gap: 10px;

                cursor: pointer;

                .aside__container_castle_white_circle {
                    width: 15px;
                    height: 15px;
                    border-radius: 50%;
                    background: white;
                    border: 1px solid white;
                }

                .aside__container_castle_white-head {
                    text-align: center;
                    font-size: 1.5rem;
                }

                .aside__container_castle_white-input {
                    transform: scale(1.5);
                }

            }

            .aside__container_castle_black {

                display: flex;
                align-items: center;
                justify-content: center;
                gap: 10px;

                cursor: pointer;

                .aside__container_castle_black_circle {
                    width: 15px;
                    height: 15px;
                    border-radius: 50%;
                    background: black;
                    border: 1px solid white;
                }

                .aside__container_castle_black-head {
                    text-align: center;
                    font-size: 1.5rem;
                }

                .aside__container_castle_black-input {
                    transform: scale(1.5);
                }

            }
        }

        .aside__container_passant {

            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            .aside__container_passant-head {
                background: var(--second-background);
                text-align: center;
                font-size: 1.5rem;
                min-width: 150px;
            }

            .aside__container_passant-input {
                font-size: 1rem;
                height: 35px;
                border: none;
                text-align: center;
                width: clamp(100px, 60%, 450px);
            }
        }
    }

    .black {
        background: black;
    }

    .white {
        background: white;
    }

    .selected {
        background: #749454;
        color: #749454;
        opacity: 0.9;
    }
</style>
