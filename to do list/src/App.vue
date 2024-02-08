<script setup>
import { ref } from 'vue'
import Contenido from './components/Contenido.vue';
import Titulos from './components/Titulos.vue';
const listaTareas = ref([])
const listaTareasOld = ref([])

//Nuevo requisito "que se use 'emit'": Se ha creado esta función que usa 'emit' desde el componente 'Contenido'
function funcionQueBorraTodasLasTareas(){
  localStorage.clear();
  alert("Se han borrado todas las tareas")
  location.reload()
}
</script>

<template>
  <main>
    <div class="container">
      <Titulos titular="Aplicación para añadir tareas"></Titulos>
      <Contenido @eventoEmitido="funcionQueBorraTodasLasTareas"></Contenido>
    </div>
  </main>
</template>

<style scope>
html,
main {
  background-color: black;
}
</style>