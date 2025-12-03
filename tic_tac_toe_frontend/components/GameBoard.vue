<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

type Cell = 'X' | 'O' | ''
type Player = 'X' | 'O'

const board = ref<Cell[]>(Array(9).fill(''))
const currentPlayer = ref<Player>('X')
const winner = ref<Player | ''>('')
const isDraw = ref(false)
const gameOver = computed(() => winner.value !== '' || isDraw.value)

const statusMessage = computed(() => {
  if (winner.value) return `Winner: ${winner.value} 🎉`
  if (isDraw.value) return "It's a draw 🤝"
  return `Next player: ${currentPlayer.value}`
})

function calculateWinner(b: Cell[]): Player | '' {
  const lines = [
    [0, 1, 2], [3, 4, 5], [6, 7, 8], // rows
    [0, 3, 6], [1, 4, 7], [2, 5, 8], // cols
    [0, 4, 8], [2, 4, 6],            // diagonals
  ] as const
  for (const [a, c, d] of lines) {
    if (b[a] && b[a] === b[c] && b[a] === b[d]) return b[a] as Player
  }
  return ''
}

function checkGameState() {
  const w = calculateWinner(board.value)
  winner.value = w
  if (!w && board.value.every(c => c !== '')) {
    isDraw.value = true
  }
}

function togglePlayer() {
  currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
}

function playMove(index: number) {
  if (gameOver.value) return
  if (board.value[index] !== '') return
  board.value[index] = currentPlayer.value
  checkGameState()
  if (!gameOver.value) togglePlayer()
}

function resetGame() {
  board.value = Array(9).fill('')
  currentPlayer.value = 'X'
  winner.value = ''
  isDraw.value = false
  // Move focus to first cell for convenience
  focusCell(0)
}

// Accessibility: refs array to focus cells
const cellRefs = ref<Array<HTMLButtonElement | null>>([])

function focusCell(idx: number) {
  const el = cellRefs.value[idx]
  if (el) el.focus()
}

// Keyboard handlers: Enter/Space plays; Arrows navigate grid; Home/End jump
function onCellKeydown(e: KeyboardEvent, idx: number) {
  const key = e.key
  const col = idx % 3
  const row = Math.floor(idx / 3)
  if (key === 'Enter' || key === ' ') {
    e.preventDefault()
    playMove(idx)
    return
  }
  // Navigation
  if (key === 'ArrowRight') {
    e.preventDefault()
    focusCell((row * 3) + ((col + 1) % 3))
  } else if (key === 'ArrowLeft') {
    e.preventDefault()
    focusCell((row * 3) + ((col + 2) % 3))
  } else if (key === 'ArrowDown') {
    e.preventDefault()
    focusCell((((row + 1) % 3) * 3) + col)
  } else if (key === 'ArrowUp') {
    e.preventDefault()
    focusCell((((row + 2) % 3) * 3) + col)
  } else if (key === 'Home') {
    e.preventDefault()
    focusCell(0)
  } else if (key === 'End') {
    e.preventDefault()
    focusCell(8)
  }
}

const headerAnnouncement = ref('')

onMounted(() => {
  // Initial focus
  focusCell(0)
})
</script>

<template>
  <section
    class="ttt-wrapper"
    aria-label="Two-player Tic Tac Toe game"
  >
    <div class="ttt-panel surface shadow">
      <div class="ttt-status">
        <div class="ttt-current">
          <span class="eyebrow">Current Player</span>
          <div class="player-chip" :class="currentPlayer">
            {{ currentPlayer }}
          </div>
        </div>
        <div
          class="ttt-message"
          role="status"
          aria-live="polite"
        >
          {{ statusMessage }}
        </div>
      </div>

      <div
        class="ttt-board"
        role="grid"
        aria-label="Tic Tac Toe board"
      >
        <button
          v-for="(cell, i) in board"
          :key="i"
          ref="cellRefs"
          class="ttt-cell"
          role="gridcell"
          :aria-label="cell ? `Cell ${i + 1}, ${cell}` : `Cell ${i + 1}, empty`"
          :aria-disabled="gameOver || !!cell"
          :disabled="gameOver || !!cell"
          @click="playMove(i)"
          @keydown="onCellKeydown($event, i)"
        >
          <span
            class="mark"
            :class="cell"
            aria-hidden="true"
          >
            {{ cell }}
          </span>
        </button>
      </div>

      <div class="ttt-controls">
        <button
          class="btn-primary reset-btn"
          type="button"
          @click="resetGame"
        >
          Reset Game
        </button>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* Color tokens based on Ocean Professional theme (provided) */
:root {
  --ocean-primary: #2563EB;
  --ocean-secondary: #F59E0B;
  --ocean-success: #F59E0B;
  --ocean-error: #EF4444;
  --ocean-bg: #0b1220; /* use dark canvas for consistency with existing theme */
  --ocean-surface: #111522;
  --ocean-surface-2: #0f1421;
  --ocean-text: #E6EDF3;
  --ocean-muted: #9CA3AF;
  --ocean-border: #23324d;
}

/* Wrapper centers the panel and provides gradient ambience */
.ttt-wrapper {
  display: grid;
  justify-items: center;
  align-items: center;
  min-height: 60vh;
  padding: 12px;
  background: linear-gradient(135deg, rgba(37, 99, 235, 0.10), rgba(243, 244, 246, 0.05));
  border-radius: 16px;
}

/* Panel surface */
.ttt-panel.surface {
  width: min(760px, 94vw);
  background: var(--ocean-surface);
  border: 1px solid var(--ocean-border);
  border-radius: 20px;
  padding: 20px;
  backdrop-filter: blur(8px);
}

.shadow {
  box-shadow: 0 20px 50px rgba(0,0,0,0.35);
}

/* Status row */
.ttt-status {
  display: grid;
  grid-template-columns: auto 1fr;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
}
.ttt-current {
  display: grid;
  gap: 6px;
  align-content: start;
}
.player-chip {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 44px;
  height: 36px;
  padding: 0 12px;
  border-radius: 999px;
  font-weight: 800;
  letter-spacing: 0.06em;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.10);
  color: var(--ocean-text);
}
.player-chip.X { box-shadow: 0 6px 18px rgba(37, 99, 235, 0.25); border-color: rgba(37, 99, 235, 0.45); }
.player-chip.O { box-shadow: 0 6px 18px rgba(245, 158, 11, 0.25); border-color: rgba(245, 158, 11, 0.45); }

.ttt-message {
  justify-self: end;
  font-weight: 700;
  color: var(--ocean-text);
  padding: 10px 14px;
  border-radius: 12px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
}

/* Board grid */
.ttt-board {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  background: var(--ocean-surface-2);
  border: 1px solid var(--ocean-border);
  border-radius: 16px;
  padding: 12px;
  margin-top: 6px;
}

/* Cells */
.ttt-cell {
  aspect-ratio: 1 / 1;
  border-radius: 16px;
  border: 1px solid var(--ocean-border);
  background: rgba(255,255,255,0.04);
  color: var(--ocean-text);
  font-size: clamp(28px, 7.2vw, 48px);
  font-weight: 900;
  display: grid;
  place-items: center;
  cursor: pointer;
  transition: transform 140ms ease, box-shadow 180ms ease, background 180ms ease, border-color 180ms ease;
  outline: none;
  position: relative;
  text-shadow: 0 2px 10px rgba(0,0,0,0.35);
}
.ttt-cell:hover:not(:disabled) {
  transform: translateY(-1px);
  border-color: rgba(255,255,255,0.18);
  background: rgba(255,255,255,0.06);
  box-shadow: 0 10px 24px rgba(0,0,0,0.35);
}
.ttt-cell:active:not(:disabled) {
  transform: translateY(0);
  box-shadow: none;
}
.ttt-cell:focus-visible {
  outline: 2px solid #8B9FFB;
  outline-offset: 2px;
}
.ttt-cell:disabled {
  cursor: not-allowed;
  opacity: 0.98;
}

/* Marks */
.mark {
  display: inline-block;
  line-height: 1;
  transform: translateZ(0);
}
.mark.X {
  color: var(--ocean-primary);
  text-shadow: 0 6px 26px rgba(37, 99, 235, 0.35);
}
.mark.O {
  color: var(--ocean-secondary);
  text-shadow: 0 6px 26px rgba(245, 158, 11, 0.35);
}

/* Controls */
.ttt-controls {
  display: flex;
  justify-content: center;
  margin-top: 16px;
}
.reset-btn {
  padding: 12px 18px;
  border-radius: 12px;
}

/* Responsive */
@media (max-width: 640px) {
  .ttt-status {
    grid-template-columns: 1fr;
    gap: 8px;
  }
  .ttt-message {
    justify-self: start;
  }
}
</style>
