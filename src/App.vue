
<script setup>
import { ref } from 'vue'

import LobbyView from './views/LobbyView.vue'
import WordRevealView from './views/WordRevealView.vue'
import GameView from './views/GameView.vue'

const jugadoresRegistrados = ref([])
const palabraActual = ref('')
const etapa = ref('lobby')

function iniciarPartida(jugadores) {
  jugadoresRegistrados.value = jugadores
  etapa.value = 'palabra'
}

function recibirPalabra(palabra) {
  palabraActual.value = palabra
  etapa.value = 'juego'
}
</script>

<template>
  <main>
    <LobbyView
      v-if="etapa === 'lobby'"
      @start-game="iniciarPartida"
    />

    <WordRevealView
      v-else-if="etapa === 'palabra'"
      @word-selected="recibirPalabra"
    />

    <GameView
      v-else-if="etapa === 'juego'"
      :jugadores="jugadoresRegistrados"
      :palabra="palabraActual"
    />
  </main>
</template>
