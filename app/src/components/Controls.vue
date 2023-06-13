<template>
  <div class="controls">
    <div class="numbers">
      <button
        v-for="option in options"
        :key="option"
        :class="[
          'tile option',
          {
            selected: option === selectedOption,
            completed: optionsCompleted.includes(option),
          },
        ]"
        :disabled="optionsCompleted.includes(option)"
        @click="handleOption(option)"
      >
        <span>{{ option }}</span>
      </button>
    </div>
    <div class="actions">
      <button
        :class="['action', { selected: selectedOption === '10' }]"
        @click="handleOption('10')"
      >
        <span class="material-symbols-outlined"> delete </span>
      </button>
      <button
        :class="['action', { selected: hintMode }]"
        @click="handleOption('11')"
      >
        <span class="material-symbols-outlined"> app_registration </span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, toRefs } from 'vue'
const options = ['1', '2', '3', '4', '5', '6', '7', '8', '9']

const props = defineProps(['selectedOption', 'hintMode', 'optionsCompleted'])

const { selectedOption, hintMode, optionsCompleted } = toRefs(props)

const emit = defineEmits(['selectedOption', 'changeHintMode'])

const handleOption = (option) => {
  if (optionsCompleted.value.includes(option)) return
  if (option === '11') {
    // hintMode.value = !hintMode.value
    emit('changeHintMode')
    return
  }
  emit('selectedOption', option)
}
</script>

<style lang="scss" scoped>
.controls {
  margin: 1rem;
  display: grid;
  grid-template-columns: 75% 25%;
}
.numbers {
  display: grid;
  grid-template-columns: repeat(5, minmax(30px, 50px));
  line-height: 1;
  gap: 5px;
  justify-content: space-between;
}
.option {
  border: 2px solid var(--clr-primary);
  aspect-ratio: 1;
  background: none;
  color: #fff;
  font-family: inherit;
  padding: 0;
  &.selected {
    border-color: red;
    font-size: 1.75rem;
  }
  &.completed {
    background-color: var(--clr-primary);
    opacity: 0.5;
  }
}
.actions {
  display: flex;
  flex-direction: column;
  line-height: 1;
  gap: 5px;
  margin-left: 1rem;
}
.action {
  background-color: transparent;
  color: #fff;
  flex: 1;
  max-height: 50px;
  border: 2px solid var(--clr-primary);
  display: grid;
  place-items: center;
  cursor: pointer;
  &.selected {
    border-color: red;
  }

  .material-symbols-outlined {
    font-size: 2.25rem;
  }
}
</style>
