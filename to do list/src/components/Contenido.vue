<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import Titulos from './Titulos.vue';

//Nuevo requisito "que se use 'emit'": Se ha creado este 'emit' para que sea usado en 'App.vue'
const emit = defineEmits(['eventoEmitido'])
function borrarTareasTodas() {
  emit('eventoEmitido')
}

//Definición de variables
const listaTareas = ref([])
const listaTareasOld = ref([])
const nombreTarea = ref('')
const prioridadTarea = ref('Baja')
const fechaTarea = ref()
const numTareasTotal = ref('0')
let numTareasCompletadas = ref('0')
let numTareasActivas = ref('0')
let numId = ref()
const prioridad_id = ref('')  //Permitirá ordenar la lista de Alta -> Media -> Baja

//Añade la tarea
const anyadirTarea = () => {
  if (nombreTarea.value.trim() === '' || fechaTarea.value == null) {
    alert('Falta nombre de tarea o fecha')      //Alert en caso que los campos estén vacíos
  }
  else {          //Asigna un número según la prioridad
    if (prioridadTarea.value == 'Alta') {
      prioridad_id.value = 1
    } else if (prioridadTarea.value == 'Media') {
      prioridad_id.value = 2
    } else {
      prioridad_id.value = 3
    }
    numId++;
    listaTareas.value.push({      //Añade al array listaTareas
      id: numId,
      content: nombreTarea.value,
      prioridad: prioridadTarea.value,
      prioridad_id: prioridad_id.value,
      done: false,
      fecha: fechaTarea.value,
      creada: new Date().getTime()
    })
    listaTareasOld.value = listaTareas.value  //Iguala los valores de los array
  }
  localStorage.setItem('numId', numId)  //Guarda el valor 'numId'
  nombreTarea.value = ''
}

const listaTareasAsc = computed(() => listaTareas.value.sort((a, b) => {
  if (a.prioridad_id > b.prioridad_id) {
    return 1;
  }

  if (a.prioridad_id == b.prioridad_id) {
    return 0;
  }

  if (a.prioridad_id < b.prioridad_id) {
    return -1;
  }
}))

const borrarTarea = (nombreTareaId) => {
  for (let i = 0; i < listaTareasOld.value.length; i++) {
    if (listaTareasOld.value[i].id == nombreTareaId) {
      listaTareasOld.value.splice(i, 1);		//Elimina la tarea con ese index de listaTareasOld
    }
  }
  for (let i = 0; i < listaTareas.value.length; i++) {
    if (listaTareas.value[i].id == nombreTareaId) {
      listaTareas.value.splice(i, 1);		//Elimina la tarea con ese index de listaTareas
    }
  }
}

const borrarTareasCompletadas = () => {
  listaTareasOld.value = listaTareasOld.value.filter((item) => {
    return item.done == false;	//Devuelve la lista de tareas activas, con done=false;
  });
  listaTareas.value = listaTareasOld.value  //Iguala los valores de las dos listas
}

const cambiarPrioridad = (prioridadBoton, nombreTareaId) => {	//Cambia la prioridad de la tarea
  if (prioridadBoton == 'Alta') {
    prioridad_id.value = 1
  } else if (prioridadBoton == 'Media') {
    prioridad_id.value = 2
  } else {
    prioridad_id.value = 3
  }
  for (let i = 0; i < listaTareasOld.value.length; i++) {
    if (listaTareasOld.value[i].id == nombreTareaId) {      //Busca la tarea con ese id unívoco y obtiene su index
      listaTareasOld.value[i].prioridad = prioridadBoton;		//Cambia la prioridad de la tarea con ese index=i
      listaTareasOld.value[i].prioridad_id = prioridad_id.value;
    }
  }
  for (let i = 0; i < listaTareas.value.length; i++) {
    if (listaTareas.value[i].id == nombreTareaId) {     //Busca la tarea con ese id unívoco y obtiene su index
      listaTareas.value[i].prioridad = prioridadBoton;	//Cambia la prioridad de la tarea con ese index=i
      listaTareas.value[i].prioridad_id = prioridad_id.value;
    }
  }
}

const calculoNumTareasCompletadas = () => {
  numTareasCompletadas = 0;
  //Cuenta las tareas completadas, con done=true;
  for (let i = 0; i < JSON.parse(localStorage.listadoLocalOld).length; i++) {
    if (JSON.parse(localStorage.listadoLocalOld)[i]['done'] == true) {
      numTareasCompletadas++;
    }
  }
}

const calculoNumTareasActivas = () => {
  numTareasActivas = 0;
  //Cuenta las tareas activas, con done=false;
  for (let i = 0; i < JSON.parse(localStorage.listadoLocalOld).length; i++) {
    if (JSON.parse(localStorage.listadoLocalOld)[i]['done'] == false) {
      numTareasActivas++;
    }
  }
}

const filtrarNombreTarea = (nombreTarea) => {	//Muestra solo las tareas que coinciden con la búsqueda
  if (localStorage.listadoLocal) {
    listaTareas.value = listaTareas.value.filter((item) => {
      return item.content.includes(nombreTarea);
    });
  }
  nombreTarea = null;
}

const getTareasTodas = () => {		//Muestra todas las tareas almacenadas
  if (localStorage.listadoLocal) {
    listaTareas.value = listaTareasOld.value
    listaTareas.value = listaTareasAsc.value
  }
  nombreTarea.value = null;
}

const getTareasCompletadas = () => {	//Muestra solo las tareas completadas
  if (localStorage.listadoLocal) {
    listaTareas.value = JSON.parse(localStorage.getItem("listadoLocalOld"));
    listaTareas.value = listaTareasAsc.value.filter((item) => {
      return item.done;		//Devuelve las tareas completadas, con done=true
    });
  }
}

const getTareasActivas = () => {	//Muestra solo las tareas completadas
  if (localStorage.listadoLocal) {
    listaTareas.value = JSON.parse(localStorage.getItem("listadoLocalOld"));
    listaTareas.value = listaTareasAsc.value.filter((item) => {
      return !item.done;		//Devuelve las tareas completadas, con done=true
    });
  }
}

watch(listaTareasOld, newVal => { //Observa cuando cambia la lista listaTareasOld
  localStorage.setItem('listadoLocalOld', JSON.stringify(newVal))
  numTareasTotal.value = JSON.parse(localStorage.listadoLocalOld).length
  calculoNumTareasCompletadas()
  calculoNumTareasActivas();
}, { deep: true }
)

watch(listaTareas, newVal => {  //Observa cuando cambia la lista listaTareas
  localStorage.setItem('listadoLocal', JSON.stringify(newVal))
}, { deep: true }
)

onMounted(() => {
  if (!localStorage.listadoLocal) {
    numId = -1;       //Inicializa el 'numId' a -1
    localStorage.setItem('numId', -1)   //Guarda el valor 'numId' en localStorage
  } else {
    numId = localStorage.getItem('numId')   //Recupera el valor de 'numId'
  }
  listaTareas.value = JSON.parse(localStorage.getItem('listadoLocal')) || []  //Carga los valores en listaTareas
  listaTareasOld.value = JSON.parse(localStorage.getItem('listadoLocalOld')) || []  //Carga los valores en listaTareasOld

  if (localStorage.listadoLocal) {
    numTareasTotal.value = JSON.parse(localStorage.listadoLocal).length	//Recupera el número de tareas total
    calculoNumTareasCompletadas();    //Llama a la función que calcula las tareas completadas
    calculoNumTareasActivas();        //Llama a la función que calcula las tareas activas
    if (listaTareas.value.length == listaTareasOld.value.length) {
      listaTareas.value = listaTareasOld.value  //Iguala los valores de listaTareasOld a listaTareas
    }
  }
})
</script>

<template>


  <div class="item">
    <main class="app">
      <section class="crear-lista">
        <div class="row gx-0">
          <!-- Intrucciones de la aplicación -->
          <div class="col-8" id="instrucciones">
            <h5> Instrucciones de uso</h5>
            <ol>
              <li>Escribe el nombre de la tarea y elige una fecha</li>
              <li>Selecciona prioridad</li>
              <li>Pulsa 'Añadir'</li>
              <li>Para filtrar tareas escribe en 'Nombre de la tarea' y pulsa <button
                  v-on:click="filtrarNombreTarea(nombreTarea)" id="filtrar"
                  class="btn btn-outline-dark btn-sm">Filtrar</button></li>
              <li class="atencion"><span class="importante">IMPORTANTE:</span> Para añadir tareas o marcar como
                completada pulsa
                antes <button class="mostrar-todas btn btn-info btn-sm" v-on:click="getTareasTodas">Mostrar
                  todas</button></li>
            </ol>
          </div>

          <!-- Control de tareas -->
          <div class="col-4" id="control-de-tareas">
            <h5>Control de tareas</h5>
            <div id="informacion">
              <div>Totales: <span id="tareas-total"> {{ numTareasTotal }} </span></div>
              <div>Completadas: <span id="tareas-done"> {{ numTareasCompletadas }} </span></div>
              <div>Activas: <span id="tareas-activas"> {{ numTareasActivas }} </span></div>
            </div>
            <br />
            <button class="btn btn-warning" @click="borrarTareasTodas">Borrar todas las tareas</button>
          </div>
        </div>
        <Titulos titulos="Añade tareas siguiendo las instrucciones"></Titulos>
        <div id="anyadir-tareas">
          <h5>Añade o filtra tareas</h5>
          <div class="row">
            <div class="col-5">Nombre de la tarea</div>
            <div class="col-2">Fecha de creación</div>
            <div class="col-3">Prioridad</div>
            <div class="col-2"></div>
          </div>

          <form @submit.prevent="anyadirTarea">
            <div class="row">
              <div class="col-5">
                <input type="text" id="tarea" placeholder="Añade una tarea" v-model="nombreTarea">
              </div>
              <div class="col-2">
                <input type="date" id="fecha" v-model="fechaTarea">
              </div>
              <div class="col-3">
                <input type="radio" id="baja" class="baja" value="Baja" v-model="prioridadTarea" checked>
                <span class="bubble"></span>
                <label for="baja">Baja</label>
                <input type="radio" id="media" class="media" value="Media" v-model="prioridadTarea">
                <span class="bubble"></span>
                <label for="media">Media</label>
                <input type="radio" id="alta" class="alta" value="Alta" v-model="prioridadTarea">
                <span class="bubble"></span>
                <label for="alta">Alta</label>
              </div>
              <div class="col-2">
                <button type="submit" class="btn btn-outline-success">Añadir</button>
              </div>
            </div>
          </form>
        </div>
      </section>
      <Titulos titulos="Listado de tareas añadidas, pulsa los botones para diferentes vistas"></Titulos>
      <section class="seccion-tareas">
        <div id="lista-de-tareas">
          <div class="row">
            <div class="row">
              <div class="col-4">
                <h5 id="estado-de-tareas">Lista de Tareas</h5>
              </div>
              <div id="botones" class="col-8"><button v-on:click="getTareasTodas" id="todas"
                  class="btn btn-info btn-sm">Mostrar todas</button>
                <button v-on:click="getTareasCompletadas" id="completadas"
                  class="btn btn-success btn-sm">Completadas</button>
                <button v-on:click="getTareasActivas" id="activas" class="btn btn-primary btn-sm">Activas</button>
                <button v-on:click="borrarTareasCompletadas" id="borrar-completadas"
                  class="btn btn-danger  btn-sm">Borrar
                  completadas</button>
              </div>
            </div>
            <div class="col-4">Nombre de la tarea</div>
            <div class="col-2">Fecha de creación</div>
            <div class="col-2">Prioridad</div>
            <div class="col-3">Cambiar prioridad</div>
            <div class="col-1">Borrar</div>
          </div>

          <div id="lista-tareas">
            <ul>
              <li v-for="(nombreTarea) in listaTareasAsc" :key="nombreTarea.id">
                <div class="row">
                  <div class="col-4" :class="{ 'nombreTareaDone': nombreTarea.done }"> <label>
                      <input type="checkbox" v-model="nombreTarea.done" />
                    </label>
                    {{ nombreTarea.content }}
                  </div>
                  <div class="col-2" :class="{ 'nombreTareaDone': nombreTarea.done }"> {{ nombreTarea.fecha }}</div>
                  <div class="col-2" :class="{ 'nombreTareaDone': nombreTarea.done }"> {{ nombreTarea.prioridad }}</div>
                  <div class="col-3 botones-prioridad">
                    <button class="baja" value="Baja" @click="cambiarPrioridad('Baja', nombreTarea.id)">B</button>
                    <button class="media" value="Media" @click="cambiarPrioridad('Media', nombreTarea.id)">M</button>
                    <button class="alta" value="Alta" @click="cambiarPrioridad('Alta', nombreTarea.id)">A</button>
                  </div>
                  <div class="actions col-1">
                    <button class="btn btn-outline-danger btn-sm borrar"
                      @click="borrarTarea(nombreTarea.id)">Borrar</button>
                  </div>
                </div>
              </li>
            </ul>

          </div>
        </div>
      </section>

    </main>
  </div>

</template>

<style scoped>
#instrucciones {
  background-color: aquamarine;
  padding: 1% 2% 1% 2%;
  border-top-left-radius: 15px;
  border-bottom-left-radius: 15px;
  margin-bottom: 1%;
}

#control-de-tareas {
  padding: 1% 2% 1% 2%;
  background-color: lightgreen;
  border-top-right-radius: 15px;
  border-bottom-right-radius: 15px;
  margin-bottom: 1%;
}

button.mostrar-todas {
  border: 2px black solid;
}

button.mostrar-todas:hover {
  border: 2px black solid;
}

.importante {
  color: darkviolet !important;
  font-weight: 700;
}

#lista-de-tareas {
  background-color: honeydew;
  padding: 2%;
  border-radius: 15px;
}

#tarea {
  width: 100%;
}

ul li {
  list-style: none;
}

ul {
  display: inline;
}

#tareas-localstorage button,
#lista-tareas button {
  padding-right: 4px;
  padding-left: 4px;
  margin-right: 5px;
}

button.borrar {
  height: 80%;
}

.nombreTareaDone {
  text-decoration: line-through;
  color: grey !important;
}

#lista-tareas div {
  color: blue;
}

.claseActivas {
  display: none;
}

.claseCompletadas {
  display: none;
}

#lista-de-tareas button {
  border: 2px black solid;
}

#anyadir-tareas {
  background-color: beige;
  padding: 2%;
  border-radius: 15px;
  margin-bottom: 1%;
}

#tarea {
  width: 100%;
}

#fecha {
  width: 100%;
  color: darkgrey;
}

label[for='baja'],
.baja {
  color: green;
}

label[for='media'],
.media {
  color: orange;
}

label[for='alta'],
.alta {
  color: red;
}

ol li {
  font-size: 0.8em;
}
</style>
