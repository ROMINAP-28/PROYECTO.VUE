
<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  dibujos: {
    type: Array,
    required: true
  },
  jugadores: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['finish-voting'])

const turnoVotante = ref(0)
const listoParaVotar = ref(false)
const votoRegistrado = ref(false)

const votos = ref({})

// Mezclar el orden de los dibujos para ocultar su autor
const dibujosMezclados = ref(
  [...props.dibujos].sort(() => Math.random() - 0.5)
)

const votanteActual = computed(() => {
  return props.jugadores[turnoVotante.value]
})

// Cada jugador solamente puede votar por dibujos ajenos
const dibujosDisponibles = computed(() => {
  return dibujosMezclados.value.filter(
    dibujo => dibujo.jugadorId !== votanteActual.value.id
  )
})

function comenzarVotacion() {
  listoParaVotar.value = true
}

function votar(dibujo) {
  if (votoRegistrado.value) return

  const idAutor = dibujo.jugadorId

  if (votos.value[idAutor] === undefined) {
    votos.value[idAutor] = 0
  }

  votos.value[idAutor]++

  votoRegistrado.value = true
}

function siguienteVotante() {
  if (turnoVotante.value + 1 < props.jugadores.length) {
    turnoVotante.value++
    listoParaVotar.value = false
    votoRegistrado.value = false
  } else {
    emit('finish-voting', { ...votos.value })
  }
}
</script>

<template>
  <section class="galeria">

    <h1>PIXELBATTLE</h1>
    <h2>Galería de dibujos</h2>

    <div v-if="!listoParaVotar">
      <h3>Turno de votación</h3>

      <p>
        Pasa el equipo a:
        <strong>{{ votanteActual.nombre }}</strong>
      </p>

      <p>Los demás jugadores no deben mirar la pantalla.</p>

      <button @click="comenzarVotacion">
        Estoy listo para votar
      </button>
    </div>

    <div v-else-if="!votoRegistrado">

      <h3>Elige tu dibujo favorito</h3>

      <p>
        Votante {{ turnoVotante + 1 }}
        de {{ jugadores.length }}
      </p>

      <div class="tarjetas">

        <div
          v-for="(dibujo, index) in dibujosDisponibles"
          :key="dibujo.jugadorId"
          class="tarjeta"
        >
          <h3>Dibujo {{ index + 1 }}</h3>

          <div class="mini-tablero">
            <div
              v-for="(color, pixelIndex) in dibujo.pixeles"
              :key="pixelIndex"
              class="mini-pixel"
              :style="{ backgroundColor: color }"
            ></div>
          </div>

          <button @click="votar(dibujo)">
            Votar por este dibujo
          </button>
        </div>

      </div>
    </div>

    <div v-else>
      <h2>¡Voto registrado!</h2>

      <p>
        Gracias por participar,
        {{ votanteActual.nombre }}.
      </p>

      <button @click="siguienteVotante">
        {{ turnoVotante + 1 === jugadores.length
          ? 'Finalizar votación'
          : 'Siguiente jugador'
        }}
      </button>
    </div>

  </section>
</template>

<style scoped>
.galeria {
  max-width: 1000px;
  margin: 35px auto;
  padding: 20px;
  text-align: center;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b883;
  font-size: 36px;
}

h2 {
  color: #333;
}

.tarjetas {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 25px;
  margin-top: 30px;
}

.tarjeta {
  padding: 20px;
  background: #f1f8f5;
  border: 1px solid #d6e9df;
  border-radius: 12px;
}

.mini-tablero {
  display: grid;
  grid-template-columns: repeat(32, 1fr);
  width: 100%;
  max-width: 220px;
  aspect-ratio: 1;
  margin: 15px auto;
  border: 2px solid #333;
  background: white;
}

.mini-pixel {
  min-width: 0;
  min-height: 0;
}

button {
  padding: 12px 20px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
}

button:hover {
  background: #369c70;
}
</style>
