<template>
  <div class="modal-overlay">
    <div class="modal">
      <h2>You Win!!!</h2>
      <p>Time: {{ timer }}</p>
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
      <div class="actions">
        <button class="option-btn" @click="handlePlay(true)">Play again</button>
      </div>
      <button class="close-btn" @click="closeModal">
        <span class="material-symbols-outlined"> close </span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { inject } from 'vue'

const stars = inject('stars')
const timer = inject('timer')
const showModal = inject('showModal')
const emit = defineEmits(['play'])
const handlePlay = (newGame) => {
  emit('play', newGame)
}
const closeModal = () => {
  showModal.value = false
}
</script>

<style lang="scss" scoped>
.modal-overlay {
  position: absolute;
  inset: 0;
  background-color: hsl(var(--clrs-hsl), 0.5);
  display: grid;
  place-items: center;
}
.modal {
  position: relative;
  width: 100%;
  max-width: 500px;
  text-align: center;
  padding: 2rem;
  border: 2px solid var(--clr-primary);
  background-color: var(--bg-body);
  h2 {
    margin-block: 2rem;
  }

  .actions {
    margin-block: 2rem;
    display: grid;
    place-items: center;
  }
  .close-btn {
    position: absolute;
    top: 0.5rem;
    right: 0.5rem;
  }
}
</style>
