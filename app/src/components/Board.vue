<template>
  <div class="board-wrapper">
    <div class="board">
      <div
        v-for="(tile, i) in initialBoard"
        :key="i"
        :class="['tile', { error: tile.error, readonly: tile.readonly }]"
        @click="handleTileClick(tile, i)"
      >
        <transition name="bounce">
          <span v-if="tile.value != '.'">{{ tile.value }}</span>
          <div v-else class="hints">
            <span v-for="hint in tile.hints" :key="hint" class="hint">{{
              hint !== '.' ? hint : ''
            }}</span>
          </div>
        </transition>
      </div>
    </div>
    <controls
      :selected-option="selectedOption"
      :hint-mode="hintMode"
      :options-completed="optionsCompleted"
      @selected-option="selectedOption = $event"
      @change-hint-mode="hintMode = !hintMode"
    />
  </div>
</template>

<script setup>
import {
  ref,
  onMounted,
  toRefs,
  nextTick,
  onBeforeUnmount,
  watch,
  inject,
} from 'vue'

import Controls from './Controls.vue'
import SudokuToolCollection from 'sudokutoolcollection'
const props = defineProps(['settings', 'previusGame'])
const { settings, previusGame } = toRefs(props)
const hintMode = ref(false)
const selectedOption = ref('1')
const rowData = ref({
  problem: '',
  solution: '',
})
const gameOver = inject('gameOver')
const showModal = inject('showModal')
const stars = inject('stars')
const timer = inject('timer')
const errorCount = ref(0)
watch(errorCount, (val) => {
  if (val === 0) {
    stars.value.forEach((s) => (s.title = 'star'))
  }
  if (val > 0 && val <= 3) {
    stars.value[val - 1].error = true
  }
})

const sudoku = SudokuToolCollection()
const initialBoard = ref([])
const optionsCompleted = ref([])

const checkForCompletedOption = (option) => {
  const errorFiltered = initialBoard.value.filter((tile) => !tile.error)

  optionsCompleted.value = ['1', '2', '3', '4', '5', '6', '7', '8', '9'].filter(
    (opt) => {
      return errorFiltered.filter((_opt) => _opt.value === opt).length === 9
    }
  )
}

const handleTileClick = (tile, i) => {
  if (tile && tile.readonly) return
  const oldValue = tile.value
  if (selectedOption.value === '10') {
    if (hintMode.value) {
      initialBoard.value[i].hints = [
        '.',
        '.',
        '.',
        '.',
        '.',
        '.',
        '.',
        '.',
        '.',
      ]
    }
    initialBoard.value[i].error = false
    initialBoard.value[i].value = '.'

    nextTick(() => {
      if (oldValue !== '.') {
        checkForCompletedOption(tile.value)
      }
      checkRow(i)
      checkCol(i)
      checkBlock(i)
    })
    return
  }

  if (hintMode.value) {
    handleHint(tile)
    return
  }

  nextTick(() => {
    checkRow(i)
    checkCol(i)
    checkBlock(i)
  }).then(() => {
    initialBoard.value[i].error = !checkForError(i)
  })

  initialBoard.value[i].value = selectedOption.value

  nextTick(() => {
    checkForCompletedOption(selectedOption.value)
  })
  nextTick(() => {
    setLocalStorage()
  })
  checkForWin()
}

const checkForWin = () => {
  const resultBoard = initialBoard.value.map((tile) => tile.value).join('')
  // eslint-disable-next-line
  console.log('resultBoard', resultBoard)
  console.log('solutionBoard', rowData.value.solution)

  if (resultBoard === rowData.value.solution) {
    setTimer(false)
    gameOver.value = true
    showModal.value = true
  }
}
const setLocalStorage = () => {
  const gameData = {
    settings: settings.value,
    problem: initialBoard.value,
    solution: rowData.value.solution,
    errorCount: errorCount.value,
    time: timer.value,
  }
  localStorage.setItem('game-data', JSON.stringify(gameData))
}
const handleHint = (tile) => {
  let index = parseInt(selectedOption.value) - 1

  if (tile.hints.includes(selectedOption.value)) {
    tile.hints[index] = '.'
  } else {
    tile.hints[index] = selectedOption.value
  }
}

const checkForError = (i) => {
  const noError = rowData.value.solution.charAt(i) === selectedOption.value
  if (!noError && errorCount.value <= 3) {
    errorCount.value++
  }
  // eslint-disable-next-line
  console.log('errorCount.value: ', errorCount.value)

  return noError
}
const checkDuplicate = (items) => {
  let noEmptys = items.filter((i) => i.value !== '.')
  let newSet = [...new Set(noEmptys.map((c) => c.value))]
  let hasDuplicated = newSet.length !== noEmptys.length

  if (hasDuplicated) {
    let values = noEmptys.map((c) => c.value)
    let duplicates = values.filter(
      (item, index) => values.indexOf(item) !== index
    )
    items.forEach((c) => {
      let isDuplicated = duplicates.includes(c.value)
      initialBoard.value[c.initIndex].error = isDuplicated
    })
  } else {
    items.forEach((c) => {
      initialBoard.value[c.initIndex].error = false
    })
  }
}

const checkCol = (i) => {
  let tempCol = []

  let count = 0

  for (let j = i; j < initialBoard.value.length; j += 9) {
    tempCol.push(initialBoard.value[j])
    removeHint(j)
  }
  for (let h = i; h >= 0; h -= 9) {
    if (count > 0) {
      tempCol.push(initialBoard.value[h])
    }
    count++

    removeHint(h)
  }

  checkDuplicate(tempCol)
}

const checkRow = (i) => {
  const row = Math.ceil((i + 1) / 9) - 1
  const indexCero = row * 9
  const lastIndex = indexCero + 8

  let tempRow = []
  for (let j = indexCero; j <= lastIndex; j++) {
    tempRow.push(initialBoard.value[j])
    removeHint(j)
  }
  checkDuplicate(tempRow)
}

const checkBlock = (i) => {
  let tempBlock = []
  const row = Math.ceil((i + 1) / 9) - 1
  const indexCeroColMainRow = row * 9
  const col = i % 9
  const indexBlockCol = Math.floor(col / 3)
  const indexBlockRow = Math.floor(row / 3)
  const indexCeroBlockCol = indexCeroColMainRow + 3 * indexBlockCol
  const indexCeroBlockRow = indexCeroBlockCol - row * 9 + indexBlockRow * 9 * 3
  const lastIndexBlock = 9 * 2 + indexCeroBlockRow + 2

  for (let i = indexCeroBlockRow; i < lastIndexBlock; i += 9) {
    for (let j = i; j < i + 3; j++) {
      tempBlock.push(initialBoard.value[j])
      removeHint(j)
    }
  }

  checkDuplicate(tempBlock)
}
const removeHint = (i) => {
  initialBoard.value[i].hints.splice(parseInt(selectedOption.value) - 1, 1, '.')
}

const timerCounter = ref(null)
const setTimer = (init) => {
  if (init) {
    // eslint-disable-next-line
    console.log('timer.value', timer.value)

    timerCounter.value = setInterval(() => {
      timer.value++
    }, 1000)
  } else {
    clearInterval(timerCounter.value)
  }
}

const initGame = () => {
  let rowProblem, solution

  // eslint-disable-next-line
  console.log('previusGame.value', previusGame.value)

  if (previusGame.value) {
    rowProblem = previusGame.value.problem
    solution = previusGame.value.solution
    errorCount.value = previusGame.value.errorCount
    timer.value = previusGame.value.time || 0
  } else {
    errorCount.value = 0
    rowProblem = sudoku.generator.generate(settings.value.difficulty)
    solution = sudoku.solver.solve(rowProblem)
    timer.value = 0
  }
  rowData.value.problem = rowProblem
  rowData.value.solution = solution

  initialBoard.value = previusGame.value
    ? rowData.value.problem
    : rowData.value.problem.split('').map((item, i) => {
        return {
          value: item,
          readonly: item !== '.',
          error: false,
          hints: ['.', '.', '.', '.', '.', '.', '.', '.', '.'],
          initIndex: i,
        }
      })

  nextTick(() => {
    const options = ['1', '2', '3', '4', '5', '6', '7', '8', '9']
    options.forEach((opt) => {
      checkForCompletedOption(opt)
    })
  })

  setLocalStorage()
  setTimer(true)
}

defineExpose({ initGame })

onMounted(() => {
  initGame()
})
onBeforeUnmount(() => {
  setLocalStorage()
  setTimer(false)
})
</script>

<style lang="scss" scoped>
.board {
  display: grid;
  grid-template-columns: repeat(9, minmax(30px, 50px));
  line-height: 1;
  margin: 1rem;

  .tile {
    background-color: var(--bg-board);
    position: relative;
    color: var(--clr-accent);
    aspect-ratio: 1;

    &.readonly {
      color: var(--clr-primary-100);
    }
    &.error {
      color: red;
    }

    &:nth-child(n):nth-child(-n + 9) {
      border-top-width: 4px;
    }
    &:nth-child(n + 73):nth-child(-n + 81) {
      border-bottom-width: 4px;
    }
    &:nth-child(3n) {
      border-right-width: 4px;
    }
    &:nth-child(9n + 1) {
      border-left-width: 4px;
    }
    &:nth-child(n + 19):nth-child(-n + 27) {
      border-bottom-width: 4px;
    }
    &:nth-child(n + 46):nth-child(-n + 54) {
      border-bottom-width: 4px;
    }
  }
}
.hints {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  place-items: center;
  width: 100%;
  height: 100%;
}
.hint {
  font-size: 0.9rem;
  font-weight: 300;
  color: rgb(124, 124, 124);
}

.bounce-enter-active {
  animation: bounce-in 0.25s;
  position: absolute;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
  position: absolute;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
</style>
