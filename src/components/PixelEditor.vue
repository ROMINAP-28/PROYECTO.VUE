
<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['save-drawing'])

const TAMANO = 32
const BLANCO = '#ffffff'

const colores = [
  '#000000', '#ffffff', '#ff3333', '#ff9900',
  '#ffe033', '#36b96b', '#3288e8', '#8855cc',
  '#f078bd', '#8b572a', '#777777', '#00cccc',
  '#ff6699', '#99cc33', '#663300', '#cccccc'
]

const colorActual = ref('#000000')
const herramienta = ref('lapiz')
const zoom = ref(480)

const pixeles = ref(Array(TAMANO * TAMANO).fill(BLANCO))

const dibujando = ref(false)
const historial = ref([])
const futuro = ref([])

const puedeDeshacer = computed(() => historial.value.length > 0)
const puedeRehacer = computed(() => futuro.value.length > 0)

function guardarHistorial() {
  historial.value.push([...pixeles.value])

  if (historial.value.length > 50) {
    historial.value.shift()
  }

  futuro.value = []
}

function pintar(indice) {
  pixeles.value[indice] = herramienta.value === 'borrador'
    ? BLANCO
    : colorActual.value
}

// Pintar un píxel
function comenzarDibujo(indice) {
  if (herramienta.value === 'relleno') {
    guardarHistorial()
    rellenar(indice)
    return
  }

  guardarHistorial()
  dibujando.value = true
  pintar(indice)
}

// Pintar mientras arrastramos
function continuarDibujo(indice) {
  if (dibujando.value) {
    pintar(indice)
  }
}

// Detener el dibujo
function detenerDibujo() {
  dibujando.value = false
}

// Rellenar una zona del mismo color
function rellenar(indice) {
  const colorOriginal = pixeles.value[indice]
  const colorNuevo = colorActual.value

  if (colorOriginal === colorNuevo) return

  const pendientes = [indice]
  const visitados = new Set()

  while (pendientes.length > 0) {
    const actual = pendientes.pop()

    if (visitados.has(actual)) continue
    if (pixeles.value[actual] !== colorOriginal) continue

    visitados.add(actual)
    pixeles.value[actual] = colorNuevo

    const fila = Math.floor(actual / TAMANO)
    const columna = actual % TAMANO

    if (fila > 0) pendientes.push(actual - TAMANO)
    if (fila < TAMANO - 1) pendientes.push(actual + TAMANO)
    if (columna > 0) pendientes.push(actual - 1)
    if (columna < TAMANO - 1) pendientes.push(actual + 1)
  }
}

// Deshacer
function deshacer() {
  if (!puedeDeshacer.value) return

  futuro.value.push([...pixeles.value])
  pixeles.value = historial.value.pop()
}

// Rehacer
function rehacer() {
  if (!puedeRehacer.value) return

  historial.value.push([...pixeles.value])
  pixeles.value = futuro.value.pop()
}

// Limpiar tablero
function limpiar() {
  if (pixeles.value.every(color => color === BLANCO)) return

  guardarHistorial()
  pixeles.value = Array(TAMANO * TAMANO).fill(BLANCO)
}

// Guardar dibujo
function guardarDibujo() {
  detenerDibujo()

  emit('save-drawing', [...pixeles.value])
}

// Liberar el mouse incluso si sale de la cuadrícula
onMounted(() => {
  window.addEventListener('pointerup', detenerDibujo)
})

onUnmounted(() => {
  window.removeEventListener('pointerup', detenerDibujo)
})

function moverSobreTablero(event) {
  if (!dibujando.value || event.buttons !== 1) return

  const pixel = event.target.closest('.pixel')

  if (pixel) {
    const indice = Number(pixel.dataset.index)
    continuarDibujo(indice)
  }
}
</script>

<template>
  <section class="editor">

    <header class="encabezado">
      <h2>PIXELBATTLE</h2>
      <p>Editor de pixel art · 32 × 32</p>
    </header>

    <!-- Herramientas -->
    <div class="herramientas">

      <button
        :class="{ activa: herramienta === 'lapiz' }"
        @click="herramienta = 'lapiz'"
      >
        ✏️ Lápiz
      </button>

      <button
        :class="{ activa: herramienta === 'borrador' }"
        @click="herramienta = 'borrador'"
      >
        🧽 Borrador
      </button>

      <button
        :class="{ activa: herramienta === 'relleno' }"
        @click="herramienta = 'relleno'"
      >
        🪣 Relleno
      </button>

      <button
        :disabled="!puedeDeshacer"
        @click="deshacer"
      >
        ↶ Deshacer
      </button>

      <button
        :disabled="!puedeRehacer"
        @click="rehacer"
      >
        ↷ Rehacer
      </button>

      <button @click="limpiar">
        Limpiar
      </button>

    </div>

    <!-- Selector de colores -->
    <div class="seccion-colores">
      <h3>Paleta de colores</h3>

      <div class="paleta">
        <button
          v-for="color in colores"
          :key="color"
          :style="{ backgroundColor: color }"
          :class="{ seleccionado: colorActual === color }"
          @click="colorActual = color"
          :aria-label="'Seleccionar ' + color"
        ></button>
      </div>

      <label class="color-personalizado">
        Elegir otro color:
        <input v-model="colorActual" type="color" />
      </label>
    </div>

    <!-- Tablero -->
    <div class="zona-tablero">

      <div
        class="cuadricula"
        :style="{
            width: zoom + 'px',
            height: zoom + 'px'
        }"
        @pointermove="moverSobreTablero"
        >
        <div
        v-for="(color, indice) in pixeles"
        :key="indice"
        class="pixel"
        :data-index="indice"
        :style="{ backgroundColor: color }"
        @pointerdown.prevent="comenzarDibujo(indice)"
        ></div>
      </div>

    </div>

    <!-- Zoom -->
    <div class="control-zoom">
      <label>Zoom: {{ zoom }} px</label>

      <input
        v-model.number="zoom"
        type="range"
        min="320"
        max="640"
        step="32"
      />
    </div>

    <button class="guardar" @click="guardarDibujo">
      Finalizar dibujo
    </button>

  </section>
</template>

<style scoped>
.editor {
  max-width: 850px;
  margin: 30px auto;
  padding: 25px;
  font-family: Arial, sans-serif;
  text-align: center;
}

.encabezado h2 {
  color: #42b883;
  font-size: 30px;
  margin-bottom: 5px;
}

.encabezado p {
  color: #777;
}

.herramientas {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 8px;
  margin: 25px 0;
}

.herramientas button {
  padding: 10px 13px;
  background: #eeeeee;
  border: 1px solid #ddd;
  border-radius: 7px;
  cursor: pointer;
}

.herramientas button.activa {
  background: #42b883;
  color: white;
}

.herramientas button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.seccion-colores {
  margin-bottom: 25px;
}

.seccion-colores h3 {
  font-size: 16px;
}

.paleta {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 9px;
  margin: 15px 0;
}

.paleta button {
  width: 32px;
  height: 32px;
  border: 2px solid #ccc;
  border-radius: 5px;
  cursor: pointer;
}

.paleta button.seleccionado {
  outline: 3px solid #42b883;
  outline-offset: 2px;
}

.color-personalizado {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  font-size: 14px;
}

.color-personalizado input {
  width: 45px;
  height: 30px;
  cursor: pointer;
}

.zona-tablero {
  display: flex;
  justify-content: center;
  overflow: auto;
  padding: 10px;
}

.cuadricula {
  display: grid;
  grid-template-columns: repeat(32, 1fr);
  grid-template-rows: repeat(32, 1fr);
  border: 3px solid #333;
  background: white;
  flex-shrink: 0;
  touch-action: none;
  user-select: none;
}

.pixel {
  border: 0.3px solid #ddd;
}

.control-zoom {
  margin: 20px auto;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
}

.control-zoom input {
  width: 180px;
}

.guardar {
  padding: 13px 30px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
}

.guardar:hover {
  background: #369c70;
}
</style>
