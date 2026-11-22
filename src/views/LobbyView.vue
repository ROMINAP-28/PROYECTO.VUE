
<script setup>
import { ref, computed } from 'vue'

// Comunicación con el componente principal
const emit = defineEmits(['start-game'])

// Lista inicial de jugadores
const jugadores = ref([
  { id: 1, nombre: '' },
  { id: 2, nombre: '' }
])

let siguienteId = 3

// Agregar un jugador
function agregarJugador() {
  if (jugadores.value.length < 4) {
    jugadores.value.push({
      id: siguienteId++,
      nombre: ''
    })
  }
}

// Eliminar un jugador
function eliminarJugador(id) {
  if (jugadores.value.length > 2) {
    jugadores.value = jugadores.value.filter(
      jugador => jugador.id !== id
    )
  }
}

// Validar nombres
const puedeComenzar = computed(() => {
  return jugadores.value.every(
    jugador => jugador.nombre.trim().length >= 2
  )
})

// Enviar jugadores al componente principal
function comenzarPartida() {
  if (puedeComenzar.value) {
    emit('start-game', jugadores.value)
  }
}
</script>

<template>
  <section class="lobby">
    <h1>PIXELBATTLE</h1>
    <h2>Registro de jugadores</h2>

    <p class="descripcion">
      Ingresa los nombres de los participantes.
      ¡Que comience la competencia!
    </p>

    <div class="jugadores">
      <div
        v-for="(jugador, index) in jugadores"
        :key="jugador.id"
        class="jugador"
      >
        <label>Jugador {{ index + 1 }}</label>

        <div class="campo">
          <input
            v-model="jugador.nombre"
            type="text"
            :placeholder="'Nombre del jugador ' + (index + 1)"
            maxlength="20"
          />

          <button
            v-if="jugadores.length > 2"
            class="eliminar"
            @click="eliminarJugador(jugador.id)"
          >
            X
          </button>
        </div>
      </div>
    </div>

    <button
      v-if="jugadores.length < 4"
      class="agregar"
      @click="agregarJugador"
    >
      + Agregar jugador
    </button>

    <button
      class="comenzar"
      :disabled="!puedeComenzar"
      @click="comenzarPartida"
    >
      Comenzar partida
    </button>

    <p class="nota">
      Mínimo 2 jugadores · Máximo 4 jugadores
    </p>
  </section>
</template>

<style scoped>
.lobby {
  max-width: 550px;
  margin: 50px auto;
  padding: 35px;
  text-align: center;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b883;
  font-size: 42px;
}

h2 {
  color: #333;
}

.descripcion {
  color: #666;
  margin-bottom: 30px;
}

.jugadores {
  text-align: left;
}

.jugador {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

.campo {
  display: flex;
  gap: 10px;
}

input {
  flex: 1;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 7px;
  font-size: 15px;
}

button {
  padding: 12px 20px;
  border: none;
  border-radius: 7px;
  cursor: pointer;
  font-size: 15px;
}

.agregar {
  background: #e5f7ee;
  color: #269b68;
  width: 100%;
  margin-bottom: 15px;
}

.comenzar {
  background: #42b883;
  color: white;
  width: 100%;
}

.comenzar:disabled {
  background: #aaa;
  cursor: not-allowed;
}

.eliminar {
  background: #ffe5e5;
  color: #d33;
}

.nota {
  color: #888;
  font-size: 13px;
}
</style>
