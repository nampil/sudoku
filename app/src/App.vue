<template>
  <div class="container">
    <header class="top-header">
      <h1>N-Sudoku</h1>
      <hr />
      <div v-if="playing" class="info">
        <div class="stars">
          <span
            v-for="(star, i) in stars"
            :key="i"
            class="material-symbols-outlined"
            :style="`font-variation-settings: 'FILL' ${star.error ? 0 : 1}`"
          >
            {{ star.title }}
          </span>
        </div>
        <div class="timer">
          <span>{{ time }}</span>
        </div>
        <div>
          <button class="close-btn" @click="handleBack">
            <span class="material-symbols-outlined"> close </span>
          </button>
        </div>
      </div>
    </header>
    <main class="playground">
      <home
        v-if="!playing"
        :has-previus-game="previusGame !== null"
        @play="handlePlay"
        @diff-selected="handleDiffSelected"
        @theme-selected="handleThemeSelected"
        @game-data-collected="handleGameDataCollected"
      />
      <board
        v-else="playing"
        ref="board"
        :is-new-game="isNewGame"
        :settings="settings"
        :previus-game="previusGame"
      />

      <modal v-if="showModal" @play="handlePlay" />
    </main>
  </div>
</template>
<script setup>
import { computed, onMounted, provide, ref, watch } from 'vue'
import Board from './components/Board.vue'
import Home from './components/Home.vue'
import Modal from './components/Modal.vue'
import { themes } from './constants/themes'

const settings = ref({
  difficulty: 'easy',
  theme: themes.blue,
})
provide('settings', settings)
const gameOver = ref(false)
provide('gameOver', gameOver)
const showModal = ref(false)
provide('showModal', showModal)
const previusGame = ref(null)
const isNewGame = ref(false)
const playing = ref(false)

const board = ref(null)

const stars = ref([
  { title: 'star', error: false },
  { title: 'star', error: false },
  { title: 'star', error: false },
])

const timer = ref(0)
provide('timer', timer)

const time = computed(() => {
  // Convert time difference from milliseconds to seconds
  let timeDiff = timer.value

  // Extract integer seconds that dont form a minute using %
  let seconds = Math.floor(timeDiff % 60) //ignoring uncomplete seconds (floor)

  // Pad seconds with a zero if neccessary
  let secondsAsString = seconds < 10 ? '0' + seconds : seconds + ''

  // Convert time difference from seconds to minutes using %
  timeDiff = Math.floor(timeDiff / 60)

  // Extract integer minutes that don't form an hour using %
  let minutes = timeDiff % 60 //no need to floor possible incomplete minutes, becase they've been handled as seconds

  // Pad minutes with a zero if neccessary
  let minutesAsString = minutes < 10 ? '0' + minutes : minutes + ''

  // Convert time difference from minutes to hours
  timeDiff = Math.floor(timeDiff / 60)

  // Extract integer hours that don't form a day using %
  let hours = timeDiff % 24 //no need to floor possible incomplete hours, becase they've been handled as seconds

  // Convert time difference from hours to days
  timeDiff = Math.floor(timeDiff / 24)

  // The rest of timeDiff is number of days
  let days = timeDiff

  let totalHours = hours + days * 24 // add days to hours
  let totalHoursAsString = totalHours < 10 ? '0' + totalHours : totalHours + ''

  if (totalHoursAsString === '00') {
    return minutesAsString + ':' + secondsAsString
  } else {
    return totalHoursAsString + ':' + minutesAsString + ':' + secondsAsString
  }
})

provide('stars', stars)

const handlePlay = (newGame) => {
  if (newGame) {
    stars.value = stars.value.map((s) => {
      return { title: 'star', error: false }
    })
    previusGame.value = null
  }

  if (playing.value) {
    board.value.initGame()
  }

  playing.value = true
  showModal.value = false
}

const handleDiffSelected = (option) => {
  settings.value.difficulty = option
}
const handleThemeSelected = (option) => {
  // eslint-disable-next-line
  console.log('option', option)

  settings.value.theme = themes[option]
}

function handleBack() {
  playing.value = false
}

const handleGameDataCollected = (gameData) => {
  previusGame.value = gameData
}

watch(
  () => settings.value.theme,
  (newTheme) => {
    document.documentElement.style.setProperty('--bg-body', newTheme.bgBody)
    document.documentElement.style.setProperty('--bg-board', newTheme.bgBoard)
    document.documentElement.style.setProperty(
      '--clr-primary',
      newTheme.clrPrimary
    )
    document.documentElement.style.setProperty(
      '--clr-primary-100',
      newTheme.clrPrimary100
    )
    document.documentElement.style.setProperty(
      '--clr-accent',
      newTheme.clrAccent
    )
  },
  { immediate: true }
)

onMounted(() => {
  const gameData = JSON.parse(localStorage.getItem('game-data'))
  if (gameData) {
    handleGameDataCollected(gameData)
  }
})
</script>

<style lang="scss">
:root {
  --clrs-hsl: 206, 100%, 30%;
}
*,
*::after,
*::before {
  box-sizing: border-box;
}
html {
  font-size: clamp(10px, 3.5vw, 16px);
}
body {
  font-family: Arial, Helvetica, sans-serif;
  margin: 0;
  display: grid;
  place-items: center;
  height: 100vh;
  background-color: var(--bg-body);
  color: #fff;
}
#app {
  display: grid;
  place-items: center;
  width: 100%;
  min-height: 100%;
  background-color: var(--bg-body);
}
.container {
  width: 100%;
  max-width: 1100px;
  margin-inline: auto;
  min-height: 100%;
}

.stars {
  color: hsl(54, 100%, 50%);
}

.top-header {
  width: 100%;
  padding: 1rem;
  margin-bottom: 2rem;
  h1 {
    margin: 0;
  }
}
.info {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.playground {
  display: grid;
  place-items: center;
  max-width: 480px;
  margin-inline: auto;
}
.tile {
  border: 1px solid var(--clr-primary);
  padding: 0;
  display: grid;
  place-items: center;
  font-weight: bold;
  font-size: 1.7rem;
  cursor: pointer;
  transition: all 100ms ease-in-out;
}

.close-btn {
  background-color: var(--clr-board);
  color: #fff;
  padding: 5px 5px;
  border: 1px solid var(--clr-primary);
  cursor: pointer;
  &:hover {
    background-color: var(--clr-primary);
  }
}

.option-btn {
  background-color: var(--clr-board);
  color: #fff;
  padding: 10px 15px;
  border: 1px solid var(--clr-primary);
  cursor: pointer;
  &:hover {
    background-color: var(--clr-primary);
  }
  &.active {
    background-color: var(--clr-primary);
  }
}
</style>
