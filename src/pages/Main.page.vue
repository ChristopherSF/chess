<script setup>
    import { ref } from 'vue'
    import Board from '../components/Board.component.vue'
    import Controls from '../components/Controls.component.vue'

    const FEN = ref('')
    const flip = ref(false)
    const isErasing = ref(false)
    const moveCopyPiece = ref(null)
    const castleWhiteKing = ref('')
    const castleWhiteQueen = ref('')
    const castleBlackKing = ref('')
    const castleBlackQueen = ref('')
    const passant = ref('-')
    const colorMoves = ref('w')

    const stockfishResponse = ref({})

    const onBoardChange = (newFEN) => {
        FEN.value = newFEN
        fetchStockfishAPI()
    }

    const handleFlip = () => {
        flip.value = !flip.value
    }

    const handleErase = () => {
        isErasing.value = !isErasing.value
        moveCopyPiece.value = isErasing.value ? ' ' : null
    }

    const setMoveCopyPiece = (piece) => {
        moveCopyPiece.value = piece
    }

    const setCastleWhiteKing = (value) => {
        castleWhiteKing.value = value
        fetchStockfishAPI()
    }

    const setCastleWhiteQueen = (value) => {
        castleWhiteQueen.value = value
        fetchStockfishAPI()
    }

    const setCastleBlackKing = (value) => {
        castleBlackKing.value = value
        fetchStockfishAPI()
    }

    const setCastleBlackQueen = (value) => {
        castleBlackQueen.value = value
        fetchStockfishAPI()
    }

    const setPasant = (value) => {
        passant.value = value
        fetchStockfishAPI()
    }

    const setColorMoves = (color) => {
        colorMoves.value = color
        fetchStockfishAPI()
    }

    const fetchStockfishAPI = () => {
        const castles = `${castleWhiteKing.value}${castleWhiteQueen.value}${castleBlackKing.value}${castleBlackQueen.value}`
        const c_castles = castles ? castles : '-'
        const c_fen = `${FEN.value} ${colorMoves.value} ${c_castles} ${passant.value} 0 1`
        console.log(c_fen)
        fetch(`https://stockfish.online/api/s/v2.php?fen=${c_fen}&depth=12`)
        .then(response => {
            if (!response.ok) {
            throw new Error('Network response was not ok');
            }
            return response.json();
        })
        .then(data => {
            console.log('Fetched data:', data);
            stockfishResponse.value = data
        })
        .catch(error => {
            console.error('Fetch error:', error);
        })
    }

</script>

<template>
    <main>
        <Board 
            :onBoardChange="onBoardChange"
            :FEN="FEN"
            :flip="flip"
            :moveCopyPiece="moveCopyPiece"
            :setMoveCopyPiece="setMoveCopyPiece"
            :isErasing="isErasing"
            :stockfishResponse="stockfishResponse"
        />
        <Controls
            :onBoardChange="onBoardChange"
            :FEN="FEN"
            :flip="flip"
            :moveCopyPiece="moveCopyPiece"
            :setMoveCopyPiece="setMoveCopyPiece"
            :handleFlip="handleFlip"
            :isErasing="isErasing"
            :handleErase="handleErase"
            :setCastleWhiteKing="setCastleWhiteKing"
            :setCastleWhiteQueen="setCastleWhiteQueen"
            :setCastleBlackKing="setCastleBlackKing"
            :setCastleBlackQueen="setCastleBlackQueen"
            :setPassant="setPasant"
            :setColorMoves="setColorMoves"
            :stockfishResponse="stockfishResponse"
        />
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
