
<script setup>
import { ref, computed } from 'vue'
import PixelEditor from '../components/PixelEditor.vue'
import DrawingGallery from '../components/DrawingGallery.vue'

const props = defineProps({
  jugadores: {
    type: Array,
    required: true
  },
  palabra: {
    type: String,
    required: true
  }
})

const dibujos = ref([])
const turnoActual = ref(0)
const mostrarEditor = ref(false)

const votosFinales = ref(null)

const jugadorActual = computed(() => {
  return props.jugadores[turnoActual.value]
})

const partidaTerminada = computed(() => {
  return turnoActual.value >= props.jugadores.length
})

function iniciarTurno() {
  mostrarEditor.value = true
}

function guardarDibujo(pixeles) {
  dibujos.value.push({
    jugadorId: jugadorActual.value.id,
    jugador: jugadorActual.value.nombre,
    pixeles: [...pixeles]
  })

  turnoActual.value++
  mostrarEditor.value = false
}

function finalizarVotacion(votos) {
  votosFinales.value = votos
}

const ranking = computed(() => {
  if (!votosFinales.value) return []

  return [...props.jugadores]
    .map(jugador => ({
      ...jugador,
      votos: votosFinales.value[jugador.id] || 0
    }))
    .sort((a, b) => b.votos - a.votos)
})
</script>

<template>
  <section class="game">

    <div v-if="!partidaTerminada">
      <div v-if="!mostrarEditor">
        <h1>Turno de dibujo</h1>

        <h2>{{ jugadorActual.nombre }}</h2>

        <p>
          Pasa el equipo a este jugador.
          Los demás no deben mirar la pantalla.
        </p>

        <button @click="iniciarTurno">
          Estoy listo
        </button>
      </div>

      <div v-else>
        <h2>Tu palabra es:</h2>
        <h1>{{ palabra }}</h1>

        <PixelEditor @save-drawing="guardarDibujo" />
      </div>
    </div>

    <div v-else-if="!votosFinales">
      <DrawingGallery
        :dibujos="dibujos"
        :jugadores="jugadores"
        @finish-voting="finalizarVotacion"
      />
    </div>

    <div v-else class="resultados">
      <h1>¡Resultados finales!</h1>

      <h2>Clasificación</h2>

      <div
        v-for="(jugador, index) in ranking"
        :key="jugador.id"
        class="puesto"
      >
        <span class="posicion">{{ index + 1 }}°</span>

        <strong>{{ jugador.nombre }}</strong>

        <span>{{ jugador.votos }} votos</span>
      </div>

      <div v-if="ranking.length">
        <h2>🏆 ¡{{ ranking[0].nombre }} ganó!</h2>

        <p>¡Gracias por jugar PIXELBATTLE!</p>
      </div>
    </div>

  </section>
</template>

<style scoped>
.game {
  max-width: 1000px;
  margin: 40px auto;
  padding: 20px;
  text-align: center;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b883;
}

button {
  padding: 13px 25px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

.resultados {
  max-width: 550px;
  margin: 40px auto;
}

.puesto {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 16px;
  margin: 12px 0;
  background: #e5f7ee;
  border-radius: 8px;
}

.posicion {
  font-size: 22px;
  font-weight: bold;
  color: #269b68;
}
</style>
