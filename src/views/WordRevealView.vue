
<script setup>
import { ref } from 'vue'

const emit = defineEmits(['word-selected'])

const palabras = [
  'Dragón',
  'Castillo',
  'Astronauta',
  'Pingüino',
  'Volcán',
  'Robot',
  'Dinosaurio',
  'Cohete',
  'Tiburón',
  'Helicóptero'
]

const palabraSeleccionada = ref('')
const mostrarPalabra = ref(false)

function seleccionarPalabra() {
  const indice = Math.floor(Math.random() * palabras.length)

  palabraSeleccionada.value = palabras[indice]
  mostrarPalabra.value = true
}

function comenzarRonda() {
  emit('word-selected', palabraSeleccionada.value)
}
</script>

<template>
  <section class="seleccion">
    <h1>¡Nueva ronda!</h1>

    <p>
      Todos los jugadores deberán dibujar la misma palabra.
    </p>

    <div v-if="!mostrarPalabra">
      <button @click="seleccionarPalabra">
        Seleccionar palabra
      </button>
    </div>

    <div v-else class="palabra">
      <h2>La palabra es:</h2>

      <h1>{{ palabraSeleccionada }}</h1>

      <button @click="comenzarRonda">
        Continuar
      </button>
    </div>
  </section>
</template>

<style scoped>
.seleccion {
  max-width: 550px;
  margin: 80px auto;
  text-align: center;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b883;
}

.palabra {
  margin-top: 35px;
  padding: 30px;
  background: #e5f7ee;
  border-radius: 12px;
}

button {
  margin-top: 20px;
  padding: 13px 25px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
}
</style>
