<script setup lang="ts">
// Se importan las librerías necesarias.
import Swal from 'sweetalert2';


// Se definen los modelos reactivos que se utilizarán en la vista y sus valores iniciales si es necesario (default).
const response = defineModel<object[]>('response');
const tipoActo = defineModel<string>('tipoActo');
const CRIP = defineModel<string>('CRIP', { default: '' });
const loading = defineModel<boolean>('loading', { default: false });


// Se definen las columnas que se mostrarán en las tablas de resultados.
const columnsNac = [
  { title: 'Cadena', key: 'cadena', resizable: true, width: 'auto' },
  { title: 'Nombres', key: 'peNombres', resizable: true, width: 'auto' },
  { title: 'Apellido Paterno', key: 'pePrimerapellido', resizable: true, width: 'auto' },
  { title: 'Apellido Materno', key: 'peSegundoapellido', resizable: true, width: 'auto' },
  { title: 'CURP', key: 'peCurp', resizable: true, width: 'auto' },
  { title: 'Error de Origen', key: 'otErrororigen', resizable: true, width: 'auto' }
];
const columnsMat = [
  { title: 'Cadena', key: 'cadena', width: 'auto', fixed: 'left' },
  { title: 'Contrayente 1 Nombres', key: 'p1Nombres', width: 'auto' },
  { title: 'C1 Apellido Paterno', key: 'p1Primerapellido', width: 'auto' },
  { title: 'C1 Apellido Materno', key: 'p1Segundoapellido', width: 'auto' },
  { title: 'C1 CURP', key: 'p1Curp', width: 'auto' },
  { title: 'Contrayente 2 Nombres', key: 'p2Nombres', width: 'auto' },
  { title: 'C2 Apellido Paterno', key: 'p2Primerapellido', width: 'auto' },
  { title: 'C2 Apellido Materno', key: 'p2Segundoapellido', width: 'auto' },
  { title: 'C2 CURP', key: 'p2Curp', width: 'auto' },
  { title: 'Error de Origen', key: 'otErrororigen', width: 'auto' }
];

// Se definen las opciones para el select de tipo de acto. Se deshabilitan las opciones que no están disponibles.
const options = [
  { label: 'Nacimiento', value: '1' },
  { label: 'Defunción', value: '2' },
  { label: 'Matrimonio', value: '3' },
  { label: 'Divorcio', value: '4', disabled: true },
  { label: 'Adopción', value: '5', disabled: true },
  { label: 'Reconocimiento', value: '6', disabled: true }
];

// Función para buscar la cadena en la API. Se valida que la CRIP tenga 14 caracteres y que se haya seleccionado un tipo de acto. Se muestra un mensaje de error si no se cumple alguna de las condiciones.
async function buscarPersona() {
  if (CRIP.value.length !== 14) {
    Swal.fire('La CRIP debe tener 14 caracteres sin el dígito verificador.', '', 'info');
    return;
  }
  else if (tipoActo.value == undefined) {
    Swal.fire('Selecciona un tipo de acto.', '', 'info');
    return;
  }

  // Se deshabilita el botón de búsqueda y se cambia el texto del botón a "Buscando...". Se activa la animación de loading.
  document.querySelector("#searchBtn")!.toggleAttribute('disabled');
  document.querySelector("#searchBtn span.n-button__content")!.textContent = 'Buscando...';
  loading.value = true;

  // Se define la URL de la API a la que se hará la petición dependiendo del tipo de acto seleccionado.
  let url: string = '';
  switch (tipoActo.value) {
    case '1':
      url = 'http://10.215.1.14/api/CirrTa01Napeticion/buscarcadena/';
      break;
    case '2':
      url = 'http://10.215.1.14/api/CirrTa03Depeticion/buscarcadena/';
      break;
    case '3':
      url = 'http://10.215.1.14/api/CirrTa09Mapeticion/buscarcadena/';
      break;
    default:
      loading.value = false;
      return [{ msg: 'Tipo de acto no válido.' }];
  }

  // Se hace la petición a la API y se obtiene la respuesta. Si no se encuentra la cadena, se muestra un mensaje de error. Si ocurre un error en la petición, se muestra un mensaje de error.
  await fetch(url + CRIP.value)
    .then((response) => response.json())
    .then((data) => {
      if (data.length === 0) {
        Swal.fire('No se encontraron resultados.', 'Verifique su CRIP.', 'error');
        return;
      }
      response.value = data;
    })
    .catch((error) => {
      console.error('Error:', error);
      Swal.fire('Ocurrió un error al buscar la cadena.', 'Vuelva a intentar o solicite asistencia.', 'error');
    });

  // Se habilita el botón de búsqueda y se cambia el texto del botón a "Buscar". Se desactiva la animación de loading.
  document.querySelector("#searchBtn")!.toggleAttribute('disabled');
  document.querySelector("#searchBtn span.n-button__content")!.textContent = 'Buscar';
  loading.value = false;

}
</script>

<template>
  <n-flex style="width: 100%; max-width: 2000px;">
    <n-card title="BUSCAR CADENA POR CRIP" size="huge" :segmented="{ content: true, footer: true }"
      header-style="text-align: center;" :bordered="true" style="width: 100%; min-width: 1400px;">
      <n-flex vertical :size="25" align="center">
        <n-flex style="width: 25%;">
          <n-text class="poppins-medium" style="font-size: medium;">TIPO DE ACTO</n-text>
          <n-select name="tipoActo" id="tipoActo" size="large" placeholder="Selecciona el tipo de acto"
            v-model:value="tipoActo" @update:value="response = []" :options="options"></n-select>
        </n-flex>
        <n-flex style="width: 25%;">
          <n-text class="poppins-medium" style="font-size: medium;">CRIP <n-text class="poppins-bold" type="error">SIN
              DÍGITO
              VERIFICADOR</n-text></n-text>
          <n-input type="text" size="large" placeholder="Ingresa la CRIP sin el dígito verificador" maxlength="14"
            v-model:value="CRIP"></n-input>
        </n-flex>
        <n-flex justify="center">
          <n-button id="searchBtn" type="info" :loading="loading" icon-placement="right"
            @click="buscarPersona()">Buscar</n-button>
        </n-flex>
        <n-data-table v-if="tipoActo !== undefined && ['1', '2'].includes(tipoActo)" :columns="columnsNac"
          :data="response" :single-line="false" style="min-width: 1300px;"></n-data-table>
        <n-data-table v-if="tipoActo == '3'" :columns="columnsMat" :data="response" :single-line="false"
          :scroll-x="2000" style="max-width: 1300px;"></n-data-table>
        <n-text v-if="tipoActo !== undefined && ['4', '5', '6'].includes(tipoActo)" type="error"
          style="font-size: xx-large;">NO
          DISPONIBLE</n-text>
      </n-flex>
    </n-card>
  </n-flex>
</template>

<style></style>