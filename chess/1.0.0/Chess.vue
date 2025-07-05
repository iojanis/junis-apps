<!-- @app {"title":"Chess Deluxe","icon":"icons/apps/gnome-chess.svg","width":520,"height":680,"singleWindow":true} -->
<template>
  <div class="chess-app h-full flex flex-col p-2 sm:p-4 text-foreground select-none">
    <!-- Chessboard -->
    <div class="board-wrapper flex-grow flex items-center justify-center">
      <div
        class="board-container-custom-grid aspect-square relative bg-white/50 dark:bg-black/50"
        :class="{ 'opacity-60 pointer-events-none': isGameOver && !chessGame?.in_checkmate() }"
      >
        <div class="chess-board-grid-surface w-full h-full border-2 border-stone-500 shadow-lg rounded overflow-hidden">
          <template v-for="(row, rowIndex) in boardRepresentation" :key="rowIndex">
            <div
              v-for="(squareData, colIndex) in row"
              :key="`${rowIndex}-${colIndex}`"
              class="cell-custom relative flex justify-center items-center text-3xl sm:text-4xl cursor-pointer"
              :class="getSquareClass(rowIndex, colIndex, squareData)"
              @click="handleSquareClick(getSquareName(rowIndex, colIndex))"
            >
              {{ getPieceUnicode(squareData) }}
              <span
                v-if="isPossibleMoveMarker(getSquareName(rowIndex, colIndex))"
                class="absolute w-3 h-3 sm:w-4 sm:h-4 bg-black/20 rounded-full top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 pointer-events-none"
              ></span>
            </div>
          </template>
        </div>
        <div v-if="isGameOver && chessGame?.in_checkmate()"
             class="absolute inset-0 bg-black/50 flex items-center justify-center text-white text-2xl sm:text-3xl font-bold z-10 rounded">
          Checkmate!
        </div>
      </div>
    </div>

    <!-- Game Status (Moved Down) -->
    <div class="mt-3 p-3 bg-white/50 dark:bg-black/50  rounded-lg shadow text-center min-h-[60px] flex flex-col justify-center">
      <p class="text-lg font-bold" :class="statusClass">{{ gameStatusMessage }}</p>
      <p v-if="chessGame && chessGame.in_check() && !chessGame.in_checkmate()" class="text-sm text-orange-500 font-semibold">
        {{ chessGame.turn() === 'w' ? 'White' : 'Black' }} is in Check!
      </p>
      <p v-if="lastMoveInfo && lastMoveInfo.san" class="mt-1 text-xs text-muted-foreground text-center h-4">
        Last: {{ lastMoveInfo.san }}
      </p>
    </div>

    <!-- Controls -->
    <div class="mt-4 flex flex-wrap gap-2 justify-center">
      <Button @click="resetGame" variant="secondary" class="px-4 py-2 text-base">
        <RotateCcw class="w-4 h-4 mr-2" /> New Game
      </Button>
      <Button @click="undoLastMove" variant="outline" class="px-4 py-2 text-base" :disabled="!canUndo">
        <Undo2 class="w-4 h-4 mr-2" /> Undo Move
      </Button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { Chess, KING, PAWN, KNIGHT, BISHOP, ROOK, QUEEN } from '@/Components/Chess/chess.js'
import { RotateCcw, Undo2 } from 'lucide-vue-next';

const chessGame = ref(null);
const boardRepresentation = ref([]);
const selectedSquare = ref(null);
const lastMoveInfo = ref(null);
const gameTurnTrigger = ref(0); // Reactive trigger for UI updates related to game state

const PIECE_UNICODE = {
  w: { p: '♙', r: '♖', n: '♘', b: '♗', q: '♕', k: '♔' },
  b: { p: '♟︎', r: '♜', n: '♞', b: '♝', q: '♛', k: '♚' }
};
const FILES = 'abcdefgh';

const forceUiUpdate = () => {
  gameTurnTrigger.value++;
}

const initializeGame = () => {
  try {
    chessGame.value = new Chess();
    updateBoardRepresentation();
    selectedSquare.value = null;
    lastMoveInfo.value = null;
    forceUiUpdate(); // Ensure UI reflects initial state
  } catch (e) {
    console.error("Failed to initialize chess.js:", e);
    chessGame.value = { /* fallback object matching 0.x.x API */
      board: () => Array(8).fill(null).map(() => Array(8).fill(null)),
      turn: () => 'w',
      in_check: () => false,
      in_checkmate: () => false,
      in_stalemate: () => false,
      in_draw: () => false,
      game_over: () => true,
      move: () => null,
      reset: initializeGame,
      get: () => null,
      fen: () => "Error - chess.js not loaded",
      history: () => [],
      undo: () => null,
      moves: () => [],
    };
    updateBoardRepresentation();
    forceUiUpdate();
  }
};

const updateBoardRepresentation = () => {
  if (chessGame.value && typeof chessGame.value.board === 'function') {
    const rawBoard = chessGame.value.board();
    if (!Array.isArray(rawBoard) || rawBoard.length !== 8 || !rawBoard.every(r => Array.isArray(r) && r.length === 8)) {
      console.error("CRITICAL: chessGame.value.board() did not return a valid 8x8 2D array!", JSON.parse(JSON.stringify(rawBoard)));
      boardRepresentation.value = Array(8).fill(null).map(() => Array(8).fill(null));
      return;
    }
    boardRepresentation.value = rawBoard.map(row =>
      row.map(p => (p ? { type: p.type, color: p.color } : null))
    );
  } else {
    boardRepresentation.value = Array(8).fill(null).map(() => Array(8).fill(null));
  }
};

const possibleMovesForSelectedPiece = computed(() => {
  // Access gameTurnTrigger to make this computed property reactive to game state changes
  gameTurnTrigger.value;
  if (!selectedSquare.value || !chessGame.value || typeof chessGame.value.moves !== 'function') {
    return [];
  }
  return chessGame.value.moves({ square: selectedSquare.value, verbose: true });
});

const getSquareName = (rowIndex, colIndex) => FILES[colIndex] + (8 - rowIndex);

const getPieceUnicode = (squareData) => {
  if (!squareData || !squareData.type || !squareData.color) return '';
  return PIECE_UNICODE[squareData.color]?.[squareData.type] || '';
};

const getSquareClass = (rowIndex, colIndex, squareData) => {
  // Access gameTurnTrigger to make this computed property reactive to game state changes
  gameTurnTrigger.value;
  const squareName = getSquareName(rowIndex, colIndex);
  const isLightSquare = (rowIndex + colIndex) % 2 === 0;
  let classes = ['transition-colors duration-100 ease-in-out'];

  if (lastMoveInfo.value && (squareName === lastMoveInfo.value.from || squareName === lastMoveInfo.value.to)) {
    classes.push(isLightSquare ? 'bg-lime-300/80' : 'bg-lime-500/80');
  } else {
    classes.push(isLightSquare ? 'bg-stone-200 text-stone-800' : 'bg-stone-500 text-stone-100');
  }

  if (selectedSquare.value === squareName) {
    classes.push('ring-3 ring-blue-500 ring-inset');
  }

  if (chessGame.value && chessGame.value.in_check() && squareData && squareData.type === KING && squareData.color === chessGame.value.turn()) {
    classes.push('bg-red-400/60 ring-2 ring-red-600');
  }
  return classes.join(' ');
};

const isPossibleMoveMarker = (squareName) => {
  // This computed already depends on possibleMovesForSelectedPiece, which depends on gameTurnTrigger
  return possibleMovesForSelectedPiece.value.some(move => move.to === squareName);
};

const handleSquareClick = (squareName) => {
  if (isGameOver.value || !chessGame.value || typeof chessGame.value.get !== 'function' || typeof chessGame.value.move !== 'function') return;

  const pieceOnClickedSquare = chessGame.value.get(squareName);

  if (selectedSquare.value) {
    const isTargetingOwnPiece = pieceOnClickedSquare && pieceOnClickedSquare.color === chessGame.value.turn();
    const isPossibleTarget = possibleMovesForSelectedPiece.value.some(m => m.to === squareName);

    if (isPossibleTarget && !isTargetingOwnPiece) {
      const moveAttempt = { from: selectedSquare.value, to: squareName, promotion: QUEEN };
      const result = chessGame.value.move(moveAttempt);
      if (result) {
        lastMoveInfo.value = result;
        updateBoardRepresentation();
        selectedSquare.value = null;
        forceUiUpdate(); // Trigger UI re-evaluation
      } else {
        selectedSquare.value = null;
        forceUiUpdate(); // Still trigger UI if selection changes
      }
    } else if (isTargetingOwnPiece) {
      selectedSquare.value = squareName;
      forceUiUpdate(); // Trigger UI for selection change
    } else {
      selectedSquare.value = null;
      forceUiUpdate(); // Trigger UI for deselection
    }
  } else {
    if (pieceOnClickedSquare && pieceOnClickedSquare.color === chessGame.value.turn()) {
      selectedSquare.value = squareName;
      forceUiUpdate(); // Trigger UI for new selection
    }
  }
};

const resetGame = () => {
  initializeGame(); // Calls forceUiUpdate internally
};

const canUndo = computed(() => {
  // Access gameTurnTrigger to make this computed property reactive to game state changes
  gameTurnTrigger.value;
  return chessGame.value && typeof chessGame.value.history === 'function' && chessGame.value.history().length > 0;
});

const undoLastMove = () => {
  if (!canUndo.value || typeof chessGame.value.undo !== 'function') return;

  chessGame.value.undo();
  updateBoardRepresentation();
  selectedSquare.value = null;

  const history = chessGame.value.history({ verbose: true });
  lastMoveInfo.value = history.length > 0 ? history[history.length - 1] : null;
  forceUiUpdate(); // Trigger UI re-evaluation
};

const gameStatusMessage = computed(() => {
  // Access gameTurnTrigger to make this computed property reactive to game state changes
  gameTurnTrigger.value;
  if (!chessGame.value || typeof chessGame.value.fen !== 'function') return "Loading...";
  if (chessGame.value.in_checkmate()) return `${chessGame.value.turn() === 'w' ? 'Black' : 'White'} wins!`;
  if (chessGame.value.in_stalemate()) return "Stalemate! It's a draw.";
  if (chessGame.value.in_draw()) return "Draw by rule!";
  return `${chessGame.value.turn() === 'w' ? 'White' : 'Black'} to move.`;
});

const statusClass = computed(() => {
  // Access gameTurnTrigger to make this computed property reactive to game state changes
  gameTurnTrigger.value;
  if (!chessGame.value) return '';
  if (chessGame.value.in_checkmate()) return 'text-green-600 dark:text-green-400';
  if (chessGame.value.in_stalemate() || chessGame.value.in_draw()) return 'text-yellow-600 dark:text-yellow-400';
  return 'text-foreground';
});

const isGameOver = computed(() => {
  // Access gameTurnTrigger to make this computed property reactive to game state changes
  gameTurnTrigger.value;
  return chessGame.value && typeof chessGame.value.game_over === 'function' ? chessGame.value.game_over() : true;
});

onMounted(() => {
  initializeGame();
});

// Watch selectedSquare to update possible moves when selection changes internally
watch(selectedSquare, () => {
  forceUiUpdate(); // This will re-evaluate possibleMovesForSelectedPiece
});

</script>

<style scoped>
.chess-app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.board-wrapper {
  max-width: 500px;
  width: 95vw;
  margin-left: auto;
  margin-right: auto;
}

.board-container-custom-grid {
  width: 100%;
}

.chess-board-grid-surface {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  grid-template-rows: repeat(8, 1fr);
}

.cell-custom {
  overflow: hidden;
}

.cell-custom:hover {
  background-color: rgba(0,0,0,0.05);
}
</style>
