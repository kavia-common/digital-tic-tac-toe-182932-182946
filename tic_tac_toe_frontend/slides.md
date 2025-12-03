---
# Global deck settings
theme: default
title: "Tic Tac Toe — Ocean Professional"
info: |
  A modern, accessible, two-player Tic Tac Toe game implemented in Slidev.
  - Keyboard accessible (Tab + Enter/Space)
  - Responsive and self-contained (no backend)
class: text-center
mdc: true
transition: fade
fonts:
  sans: Inter, ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica Neue, Arial
  mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace
css: |
  @import "./style.css";
---

layout: center
class: text-center
---

<div class="ttt-container">
  <div class="ttt-header">
    <h1 class="ttt-title">Tic Tac Toe</h1>
    <p class="ttt-subtitle">Two Players • X starts • Ocean Professional Theme</p>
  </div>

  <div class="ttt-game-surface">
    <!-- Game component -->
    <GameBoard />
  </div>

  <div class="ttt-footer subtle">
    • Use Tab to focus a cell • Press Space or Enter to play • Click "Reset Game" to start over
  </div>
</div>

<!-- Register local component -->
<script setup lang="ts">
import GameBoard from './components/GameBoard.vue'
</script>
