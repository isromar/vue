<script setup>
import Contenido from './components/Contenido.vue'

const notificarListaBorrada = () => {
  alert('Se han borrado todas las tareas.')
}
</script>

<template>
  <Contenido @eventoEmitido="notificarListaBorrada" />
</template>
