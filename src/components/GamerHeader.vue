
<script setup>
defineProps({
  titulo: {
    type: String,
    default: 'PIXELBATTLE'
  },
  subtitulo: {
    type: String,
    default: '¡Demuestra tu creatividad!'
  }
})
</script>

<template>
  <header class="game-header">
    <h1>{{ titulo }}</h1>
    <p>{{ subtitulo }}</p>
  </header>
</template>

<style scoped>
.game-header {
  text-align: center;
  padding: 30px;
}

h1 {
  color: #42b883;
  font-size: 48px;
}

p {
  font-size: 18px;
  color: #555;
}
</style>
