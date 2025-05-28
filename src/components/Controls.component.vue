<script setup>

    import { ref } from 'vue' 
    import { v4 as uuidv4 } from 'uuid'

    const pieces = [
        'pawn',
        'knight',
        'bishop',
        'rook',
        'queen',
        'king'
    ]

    const pieces_indexed = pieces.map(piece => ({
        value: piece,
        id: uuidv4()
    }));

    const color_pieces = ref(true)

    const changeColorPieces = () => color_pieces.value = !color_pieces.value

    //------------------------------------FORM INFO

    const color_moves = ref(true)

    const changeColorMoves = () => color_moves.value = !color_moves.value


</script>

<template>
    <aside>
        <label class="aside__container-title">Stockfish Analyzer</label>
        <div class="aside__container_analize">
            <label class="aside__container_analize-percentage">0.0</label>
            <div class="aside__container_analize_bar" o>
                <div class="aside__container_analize_bar-increment"></div>
            </div>
        </div>
        <div class="aside__container_pieces">
            <div
                class="aside__container_pieces-color"
                :class="[color_pieces ? 'white' : 'black']"
                @click="changeColorPieces"
            ></div>
            <div class="aside__container_pieces_container">
                <img v-for="p in pieces_indexed" :key="p.id" :src="`/chess-pieces/${p.value}${color_pieces ? '_w' : '_b'}.png`" alt="" class="aside__container_pieces_container-item">
            </div>
        </div>
        <div class="aside__container_editors">
            <button class="aside__container_editors-flip">Flip</button>
            <button class="aside__container_editors-clear-board">Clear Board</button>
            <button class="aside__container_editors-eraser">Eraser</button>
        </div>
        <div class="aside__container_fen">
            <label class="aside__container_fen-head">FEN</label>
            <input class="aside__container_fen-input" type="text">
        </div>
        <div class="aside__container_moves">
            <label class="aside__container_moves-head">Moves</label>
            <div
                class="aside__container_moves-color"
                :class="[color_moves ? 'white' : 'black']"
                @click="changeColorMoves"
            ></div>
        </div>
        <div class="aside__container_castle">
            <label class="aside__container_castle-head">Castle</label>
            <div class="aside__container_castle_white">
                <label class="aside__container_castle_white-head" for="white">White</label>
                <input class="aside__container_castle_white-input" type="checkbox" id="white">
            </div>
             <div class="aside__container_castle_black">
                <label class="aside__container_castle_black-head" for="black">Black</label>
                <input class="aside__container_castle_black-input" type="checkbox" id="black">
            </div>
        </div>
        <div class="aside__container_passant">
            <label class="aside__container_passant-head" for="passant">Passant</label>
            <input class="aside__container_passant-input" type="checkbox" id="passant">
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
                font-size: 1.8rem;
                border: none;
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

            }

            .aside__container_pieces_container {

                display: flex;
                align-items: center;
                justify-content: center;

                .aside__container_pieces_container-item {
                    width: 40px;
                    height: 40px;
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
            }

        }

        .aside__container_castle {
            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 20px;

            .aside__container_castle-head {

                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;

            }

            .aside__container_castle_white {

                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;

            }

            .aside__container_castle_black {

                text-align: center;
                font-size: 1.5rem;
                min-width: 100px;

            }
        }

        .aside__container_passant {
            text-align: center;
            font-size: 1.5rem;
            min-width: 100px;
        }
    }

    .black {
        background: black;
    }

    .white {
        background: white;
    }
</style>
