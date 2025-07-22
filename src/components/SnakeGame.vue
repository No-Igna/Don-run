<template>
  <div class="snake-game">
    <canvas ref="canvas" class="game-canvas"></canvas>

    <div class="controls" v-if="!isGameOver">
      <button @click="startGame" v-if="!running">Iniciar juego</button>
      <button @click="resetGame" v-if="running">Reiniciar</button>
      <p>Puntaje: {{ score }}</p>
    </div>

    <div class="game-over" v-else>
      <h3>Fin del Juego</h3>
      <p>Tu puntaje fue: {{ score }}</p>
      <button @click="startGame">Volver a jugar</button>
    </div>
  </div>
</template>

<script>
//
import { defineComponent, ref, onMounted, onBeforeUnmount } from 'vue'

export default defineComponent({
  name: 'SnakeGame',
  setup() {
  const canvas = ref(null)
  const ctx = ref(null)
  const scale = ref(1) // Escala dinámica
  const rows = 10
  const columns = 15
  const canvasWidth = ref(0)
  const canvasHeight = ref(0)

  const angleMap = {
    '0,-1': -Math.PI / 2,
    '1,0': 0,
    '0,1': Math.PI / 2,
    '-1,0': Math.PI
  }

  const snake = ref([])
  const direction = ref({ x: 1, y: 0 })
  const point = ref({})
  const score = ref(0)
  const running = ref(false)
  const isGameOver = ref(false)
  let gameInterval = null

  const quixoteImg = new Image()
  quixoteImg.src = '/alleVoy.png'

  const trailImg = new Image()
  trailImg.src = '/trail.png'

  const starImg = new Image()
  starImg.src = '/Don_Quixote_Icon.png'
  

  const updateCanvasSize = () => {
    const containerWidth = window.innerWidth * 0.9
    const containerHeight = window.innerHeight * 0.6

    const scaleX = containerWidth / columns
    const scaleY = containerHeight / rows
    scale.value = Math.floor(Math.min(scaleX, scaleY))

    canvasWidth.value = columns * scale.value
    canvasHeight.value = rows * scale.value

    if (canvas.value) {
      canvas.value.width = canvasWidth.value
      canvas.value.height = canvasHeight.value
      ctx.value = canvas.value.getContext('2d')
      if (running.value) draw()
    }
  }

  const placePoint = () => {
    let x, y
    do {
      x = Math.floor(Math.random() * columns)
      y = Math.floor(Math.random() * rows)
    } while (snake.value.some(seg => seg.x === x && seg.y === y))
    point.value = { x, y }
  }

  const changeDirection = (e) => {
    const key = e.key
    if (key === 'ArrowLeft' && direction.value.x === 0) {
      direction.value = { x: -1, y: 0 }
    } else if (key === 'ArrowUp' && direction.value.y === 0) {
      direction.value = { x: 0, y: -1 }
    } else if (key === 'ArrowRight' && direction.value.x === 0) {
      direction.value = { x: 1, y: 0 }
    } else if (key === 'ArrowDown' && direction.value.y === 0) {
      direction.value = { x: 0, y: 1 }
    }
  }

  const draw = () => {
    ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height)
    
    for (let i = 1; i < snake.value.length; i++) {
      const seg = snake.value[i]
      const nextSeg = snake.value[i - 1]
      const dx = nextSeg.x - seg.x
      const dy = nextSeg.y - seg.y
      const angle = angleMap[`${dx},${dy}`] ?? 0

      ctx.value.save()
      ctx.value.translate(seg.x * scale.value + scale.value / 2, seg.y * scale.value + scale.value / 2)
      ctx.value.rotate(angle)
      ctx.value.drawImage(trailImg, -scale.value / 2, -scale.value / 2, scale.value, scale.value)
      ctx.value.restore()
    }

    const head = snake.value[0]
    if (quixoteImg.complete) {
      const angle = angleMap[`${direction.value.x},${direction.value.y}`] ?? 0
      ctx.value.save()
      ctx.value.translate(head.x * scale.value + scale.value / 2, head.y * scale.value + scale.value / 2)
      ctx.value.rotate(angle)
      ctx.value.drawImage(quixoteImg, -scale.value / 2, -scale.value / 2, scale.value, scale.value)
      ctx.value.restore()
    }

    if (starImg.complete) {
      ctx.value.drawImage(starImg, point.value.x * scale.value, point.value.y * scale.value, scale.value, scale.value)
    }
  }

  const gameLoop = () => {
    const head = {
      x: snake.value[0].x + direction.value.x,
      y: snake.value[0].y + direction.value.y
    }

    const collided =
      head.x < 0 || head.x >= columns ||
      head.y < 0 || head.y >= rows ||
      snake.value.some(seg => seg.x === head.x && seg.y === head.y)

    if (collided) {
      endGame()
      return
    }

    snake.value.unshift(head)

    if (head.x === point.value.x && head.y === point.value.y) {
      score.value++
      placePoint()
    } else {
      snake.value.pop()
    }

    draw()
  }

  const startGame = () => {
    score.value = 0
    isGameOver.value = false
    direction.value = { x: 1, y: 0 }
    snake.value = [{ x: 5, y: 1 }]
    placePoint()
    draw()
    gameInterval = setInterval(gameLoop, 150)
    running.value = true
  }

  const resetGame = () => {
    clearInterval(gameInterval)
    running.value = false
    score.value = 0
    snake.value = []
    ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height)
  }

  const endGame = () => {
    clearInterval(gameInterval)
    running.value = false
    isGameOver.value = true
  }

  onMounted(() => {
    updateCanvasSize()
    window.addEventListener('keydown', changeDirection)
    window.addEventListener('resize', updateCanvasSize)
  })

  onBeforeUnmount(() => {
    window.removeEventListener('keydown', changeDirection)
    window.removeEventListener('resize', updateCanvasSize)
    clearInterval(gameInterval)
  })

  return {
    canvas,
    score,
    startGame,
    resetGame,
    running,
    isGameOver
  }
}
})
</script>

<style scoped>
.snake-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  padding: 2rem;
  background-color: #0a0a0a95; /* fondo oscuro heroico */
  border-radius: 16px;
  box-shadow: 0 0 10px rgba(255, 204, 0, 0.2);
  max-width: 100%;
}

.game-canvas {
  max-width: 100%;
  height: auto;
  border: 3px solid #ffcc00;
  background-color: #111;
  border-radius: 12px;
  box-shadow: 0 0 12px rgba(255, 204, 0, 0.3);
}

/* Botones y controles */
.controls,
.game-over {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  color: #fff;
  font-family: 'Segoe UI', sans-serif;
}

/* Botones */
.controls button,
.game-over button {
  background-color: #ffcc00;
  color: #111;
  border: none;
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.controls button:hover,
.game-over button:hover {
  background-color: #ffd633;
}

/* Game Over */
.game-over {
  font-size: 1.4rem;
  color: #ff6666;
  text-shadow: 0 0 6px crimson;
}
</style>
