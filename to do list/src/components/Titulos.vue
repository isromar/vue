<script setup>
// Nuevo requisito "al menos dos componentes": Se ha creado este componente  para que sea usado en otro
const encabezados = defineProps(['titular','titulos'])
</script>

<template>
    <div class="titular">
        {{ encabezados.titular }}
    </div>
    <div class="titulos">
        {{ encabezados.titulos }}
    </div>
</template>

<style>
.titular {
    font-family: 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif;
    font-size: 2em;
    font-weight: bold;
    color: burlywood;
    text-align: center;
}
.titulos {
    font-size: 1.6em;
    font-weight: 500;
    color:beige;
}


</style>