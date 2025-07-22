<script setup>
import SnakeGame from './components/SnakeGame.vue';
</script>

<template>
  <div class="background">
    <div class="grid"></div>
    <div class="shine"></div>
  </div>
  <SnakeGame />
</template>

<style scoped>
.background {
  position: fixed;
  inset: 0;
  z-index: -1;
  background-color: #0a0a0a;
  overflow: hidden;
}

.grid {
  position: absolute;
  inset: 0;
  background-image: 
    linear-gradient(to right, rgba(255,255,255,0.08) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255,255,255,0.08) 1px, transparent 1px);
  background-size: 40px 40px;
  pointer-events: none;
}

.shine {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    120deg,
    transparent 30%,
    rgba(255, 204, 0, 0.3) 50%,
    transparent 70%
  );
  animation: shine 4s linear infinite;
  mix-blend-mode: screen;
  pointer-events: none;
}

@keyframes shine {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}
</style>