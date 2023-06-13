<template>
  <div class="home">
    <h2>Welcome</h2>

    <button class="option-btn mr-2" @click="handlePlay(true)">
      Start New SUDOKU
    </button>
    <button v-if="hasPreviusGame" class="option-btn" @click="handlePlay(false)">
      Continue Last One
    </button>
    <h3>Select Level</h3>
    <div class="diff-options">
      <button
        v-for="opt in difficultyOptions"
        :key="opt"
        :class="[
          'option-btn diff-option',
          { selected: opt === settings.difficulty },
        ]"
        @click="handleSelectDiff(opt)"
      >
        {{ opt }}
      </button>
    </div>
    <h3>Select Theme</h3>
    <div class="theme-options">
      <button
        v-for="opt in themeOptions"
        :key="opt"
        :class="[
          'option-btn theme-option',
          { active: settings.theme.name === opt },
        ]"
        @click="handleSelectTheme(opt)"
      >
        {{ opt }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { inject, onMounted, toRefs } from 'vue'
import { themes } from '../constants/themes'
const emit = defineEmits(['play', 'game-data-collected'])
const settings = inject('settings')
const props = defineProps(['hasPreviusGame'])
const { hasPreviusGame } = toRefs(props)

const difficultyOptions = [
  'easy',
  'medium',
  'hard',
  'very',
  'insane',
  'inhuman',
]
const themeOptions = Object.keys(themes)
const handlePlay = (newGame) => {
  emit('play', newGame)
}
const handleSelectDiff = (option) => {
  emit('diff-selected', option)
}
const handleSelectTheme = (option) => {
  emit('theme-selected', option)
}
onMounted(() => {
  const gameData = JSON.parse(localStorage.getItem('game-data'))
  if (gameData) {
    emit('game-data-collected', gameData)
  }
})
</script>

<style lang="scss" scoped>
.mr-2 {
  margin-right: 1rem;
}

.diff-option,
.theme-option {
  text-transform: capitalize;

  &.selected {
    background-color: var(--clr-primary);
  }
}
</style>
