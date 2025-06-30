<template>
  <div class="min-h-screen flex flex-col items-center p-4 bg-black">
    <!--Notificacion-->
    <section v-if="showNotification && isInsideCampus && currentView === 'campus'"
      class="fixed top-4 left-1/2 -translate-x-1/2 z-50 w-full max-w-sm bg-yellow-100 p-3 rounded-lg shadow-lg border border-yellow-200 text-center transition-opacity duration-500 ease-out">
      <h2 class="text-xl font-bold text-gray-700 mb-2">Notificación de Estacionamientos</h2>
      <div id="notifications" class="text-gray-800">
        <NotificationSector :notifications="availableNotifications"></NotificationSector>
      </div>
    </section>

    <header class="bg-[#B08C43] p-4 flex items-center justify-between rounded-t-xl max-w-4xl w-full">
      <div class="flex-grow mr-4">
        <h1 class="text-l sm:text-3xl font-bold text-white mb-2">Bienvenido a ParkCar UFRO</h1>
        <p class="text-base sm:text-lg text-white">Encuentra tu estacionamiento ideal en el campus</p>
      </div>
      <div class="flex-shrink-0 w-24 h-24 sm:w-32 sm:h-32">
        <img src="./assets/Logo ParkCar.png" alt="Logo ParkCar" class="w-full h-full object-contain rounded-full">
      </div>
    </header>

    <div class="bg-white rounded-b-xl shadow-lg p-6 w-full max-w-4xl border border-gray-200">
      <div id="location-status"
        :class="['bg-blue-100 border-l-4 p-4 rounded-md mb-6', isInsideCampus ? 'border-blue-500 text-blue-700' : 'border-red-500 text-red-700']"
        role="alert">
        <p class="font-bold">Ubicación:</p>
        <p>{{ locationText }}</p>
      </div>

      <div class="flex flex-wrap justify-center gap-4 mb-8">
        <button @click="simulateInside"
          class="px-6 py-3 bg-indigo-600 active:bg-indigo-700 text-white font-semibold rounded-lg shadow-md transition-all duration-200 ease-in-out transform active:scale-105">
          Simular Dentro del Campus
        </button>
        <button @click="simulateOutside"
          class="px-6 py-3 bg-red-600 active:bg-red-700 text-white font-semibold rounded-lg shadow-md transition-all duration-200 ease-in-out transform active:scale-105">
          Simular Fuera del Campus
        </button>
      </div>

      <!--Seccion del Mapa del Campu-->
      <div v-if="isInsideCampus" class="bg-white rounded-b-xl shadow-lg p-2 w-full max-w-4xl border border-gray-200">
        <template v-if="currentView === 'campus'">
          <section class="mb-8 p-2 bg-gray-50 rounded-lg shadow-inner border border-gray-100 relative z-10">
            <h2 class="text-2xl font-bold text-gray-700 mb-4 text-center">Mapa de Estacionamientos del Campus</h2>
            <div class="h-96 w-full rounded-lg overflow-hidden relative z-10">
              <ParkingMap :sectors="parkingSectors" :sectorToHighlight="sectorToHighlight" :filterSectorId="null">
              </ParkingMap>
            </div>
          </section>
          <!--Sectores Disponibles-->
          <section v-if="isInsideCampus"
            class="mb-8 p-2 bg-gray-50 rounded-lg shadow-inner border border-gray-100 relative z-10">
            <h2 class="text-lg sm:text-xl font-bold text-gray-700 mb-4 text-center">Sectores para seleccionar:</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
              <div v-for="sector in parkingSectors" :key="sector.id"
                :class="['p-4 rounded-lg shadow-md flex items-center justify-between transition-all duration-200 ease-in-out cursor-pointer', getAvailabilityColorClass(sector.availability)]"
                @click="goToSectorDetail(sector.id)">
                <h3 class="text-m sm:text-lg font-semibold text-white">{{ sector.name }}</h3>
                <span class="text-white font-bold text-m sm:text-lg">{{ sector.availability }}%</span>
              </div>
            </div>
          </section>
        </template>
        <!--Seccion del Mapa de un Sector-->
        <template v-else-if="currentView === 'sectorDetail' && selectedSector">
          <h2 class="text-2xl font-bold text-gray-800 mb-4 text-center">Estacionamientos en {{ selectedSector.name }}
          </h2>

          <section class="mb-8 p-2 bg-gray-50 rounded-lg shadow-inner border border-gray-100 relative z-10">
            <h3 class="text-xl font-bold text-gray-700 mb-4 text-center">Mapa del Sector</h3>
            <div class="h-96 w-full rounded-lg overflow-hidden relative z-10">
              <ParkingMap :sectors="parkingSectors" :sectorToHighlight="sectorToHighlight"
                :filterSectorId="selectedSector.id"></ParkingMap>
            </div>
          </section>
          <!--Estacionamientos Disponibles-->
          <section class="mb-8 p-2 bg-gray-50 rounded-lg shadow-inner border border-gray-100">
            <h3 class="text-lg sm:text-xl font-bold text-gray-700 mb-4 text-center">Estacionamientos disponibles:</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div v-for="parking in selectedSector.parkings" :key="parking.id"
                class="flex items-center bg-black rounded-lg shadow-md p-4 border border-gray-200 cursor-pointer active:bg-gray-800 transition-colors"
                @click="goToParkingDetail(parking.id)">

                <div>
                  <h4 class="text-m sm:text-lg font-semibold text-white">{{ parking.name }}</h4>
                  <p class="text-white text-sm sm:text-m">Espacios disponibles:
                    <span
                      :class="{ 'font-bold': true, 'text-[#47B947]': parking.availableSpaces > 10, 'text-[#FF6F00]': parking.availableSpaces <= 10 && parking.availableSpaces > 6, 'text-[#FF0315]': parking.availableSpaces <= 6 }">
                      {{ parking.availableSpaces }} / {{ parking.totalSpaces }}
                    </span>
                  </p>
                </div>
                <img :src="parking.imageUrl" :alt="parking.name" class="w-20 h-20 rounded-full ml-auto object-cover">
              </div>
            </div>
            <button @click="goToCampusView"
              class="mb-4 mt-4 px-4 py-2 bg-blue-500 active:bg-blue-600 text-white font-semibold rounded-lg shadow-md transition-colors">
              ← Volver a Vista General
            </button>
          </section>
        </template>

        <!--Vistas de espacios de estacionamientos-->
        <template v-else-if="currentView === 'parkingDetail' && selectedParking">
          <h2 class="text-2xl font-bold text-gray-800 mb-4 text-center">Espacios en {{ selectedParking.name }}</h2>
          <p class="text-center text-gray-600 mb-6">
            Total: {{ selectedParking.totalSpaces }} | Disponibles: {{ selectedParking.availableSpaces }}
          </p>


          <h3 class="text-xl font-bold text-gray-700 mb-4 text-center">Distribución de Espacios</h3>

          <div
            class="relative w-full aspect-video bg-gray-300 rounded-lg p-2 flex flex-col justify-between items-center overflow-hidden">

            <div class="absolute bottom-1/4 left-0 w-3/4 sm:w-2xl h-2/4 bg-gray-400 z-0"></div>
            <!--Espacios Superior-->
            <div class="absolute top-2 left-2 flex flex-wrap gap-1 z-10 justify-start">
              <div v-for="space in selectedParking.spaces.filter(s => s.row === 'left')" :key="space.id" :class="[
                'w-8 h-8 sm:w-20 sm:h-20 flex items-center justify-center rounded-sm border-2 transform rotate-90 transition-all duration-100',
                space.isOccupied ? 'bg-red-400 border-red-600 cursor-not-allowed' : 'bg-gray-200 border-gray-400 cursor-pointer active:bg-gray-300',
                { 'bg-green-600 !border-green-800': selectedSpaceId === space.id } // Clase para destacar
              ]" :title="space.isOccupied ? 'Ocupado' : 'Disponible'" @click="selectSpace(space)">
              </div>
            </div>
            <!--Espacios Derecho-->
            <div
              class="absolute top-2 left-7/8 -translate-x-1/2 flex flex-col gap-1 z-10 justify-center h-[calc(100%-16px)]">
              <div v-for="space in selectedParking.spaces.filter(s => s.row === 'center')" :key="space.id" :class="[
                'w-8 h-6 sm:w-20 sm:h-20 flex items-center justify-center rounded-sm border-2 transition-all duration-100',
                space.isOccupied ? 'bg-red-400 border-red-600 cursor-not-allowed' : 'bg-gray-200 border-gray-400 cursor-pointer active:bg-gray-300',
                { 'bg-green-600 !border-green-800': selectedSpaceId === space.id } // Clase para destacar
              ]" :title="space.isOccupied ? 'Ocupado' : 'Disponible'" @click="selectSpace(space)"></div>
            </div>
            <!--Espacios Inferior-->
            <div class="absolute bottom-1 sm:bottom-3 left-2 flex flex-wrap gap-1 z-10 justify-start">
              <div v-for="space in selectedParking.spaces.filter(s => s.row === 'bottom')" :key="space.id" :class="[
                'w-8 h-8 sm:w-20 sm:h-20 flex items-center justify-center rounded-sm border-2 transform rotate-90 transition-all duration-100',
                space.isOccupied ? 'bg-red-400 border-red-600 cursor-not-allowed' : 'bg-gray-200 border-gray-400 cursor-pointer active:bg-gray-300',
                { 'bg-green-600 !border-green-800': selectedSpaceId === space.id } // Clase para destacar
              ]" :title="space.isOccupied ? 'Ocupado' : 'Disponible'" @click="selectSpace(space)">
              </div>
            </div>

          </div>

          <div class="flex justify-center gap-4 mt-6">
            <button @click="backToSectorDetail"
              class="px-4 py-2 bg-red-800 active:bg-red-700 text-white font-semibold rounded-lg shadow-md transition-colors">
              ← Volver al Sector
            </button>

            <button @click="handleContinue" :disabled="!selectedSpaceId" :class="[
              'px-6 py-3 font-semibold rounded-lg shadow-md transition-all duration-200 ease-in-out',
              selectedSpaceId ? 'bg-gray-700 active:bg-gray-600 text-white' : 'bg-gray-300 text-gray-500 cursor-not-allowed'
            ]">
              Continuar →
            </button>
          </div>
        </template>

        <template v-else-if="currentView === 'navigation' && destinationCoords">
          <ParkingNavigation :destinationCoords="destinationCoords" @back="backToParkingDetail"
            class="h-[600px] w-full" />
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import ParkingMap from './components/ParkingMap.vue';
import NotificationSector from './components/NotificationSector.vue';
import ParkingNavigation from './components/ParkingNavigation.vue';
// Simulamos la BD con un archivo JSON
import initialParkingSectorsData from './sectores.json';


const isInsideCampus = ref(false);
const locationText = ref('Esperando simulación de GPS...');
const parkingSectors = ref([]);
let updateInterval = null;

// Variables de notificacion
const showNotification = ref(false);
let notificationTimer = null;

const sectorToHighlight = ref(null);

// Carga de datos de los sectores
const initialParkingSectors = initialParkingSectorsData;

// Variables para cambiar las vistas
const currentView = ref('campus'); // 'campus', 'sectorDetail', 'parkingDetail', 'navigation'
const selectedSector = ref(null);

const selectedParking = ref(null);
const selectedSpaceId = ref(null);

const destinationCoords = ref(null);

const highlightSectorOnMap = (coords) => {
  sectorToHighlight.value = coords;
};

const getRandomAvailability = () => Math.floor(Math.random() * 101); //Para simular los datos de disponibilidad

const updateSectorAvailability = () => {
  parkingSectors.value = initialParkingSectors.map(sector => {
    const updatedParkings = (sector.parkings || []).map(parking => {
      // --- MODIFICACIÓN CLAVE AQUÍ ---
      const SPACES_PER_DEMO_ROW = 6; // Definimos el número fijo de espacios por fila para la demostración
      const DEMO_TOTAL_SPACES = SPACES_PER_DEMO_ROW * 3; // Total de espacios para la demostración (6 por 3 filas)

      const spaces = [];
      let spaceCounter = 1;

      // Generar 6 espacios para la fila izquierda
      for (let i = 0; i < SPACES_PER_DEMO_ROW; i++) {
        spaces.push({
          id: `${parking.id}-space-${spaceCounter++}`,
          isOccupied: Math.random() < 0.5, // Aleatorio para la demostración
          row: 'left'
        });
      }

      // Generar 6 espacios para la fila central
      for (let i = 0; i < SPACES_PER_DEMO_ROW; i++) {
        spaces.push({
          id: `${parking.id}-space-${spaceCounter++}`,
          isOccupied: Math.random() < 0.5, // Aleatorio para la demostración
          row: 'center'
        });
      }

      // Generar 6 espacios para la fila inferior
      for (let i = 0; i < SPACES_PER_DEMO_ROW; i++) {
        spaces.push({
          id: `${parking.id}-space-${spaceCounter++}`,
          isOccupied: Math.random() < 0.5, // Aleatorio para la demostración
          row: 'bottom'
        });
      }
      // --- FIN DE LA MODIFICACIÓN CLAVE ---


      const availableSpaces = spaces.filter(s => !s.isOccupied).length;

      return {
        ...parking,
        // Ahora, el `totalSpaces` y `availableSpaces` del parking reflejarán la demo
        totalSpaces: DEMO_TOTAL_SPACES, // Sobreescribir el total para la demo
        availableSpaces: availableSpaces,
        spaces: spaces,
        destinationCoords: parking.coords
      };
    });

    // Calcular la disponibilidad del sector en base a los datos de la demo
    const totalAvailableSector = updatedParkings.reduce((sum, p) => sum + p.availableSpaces, 0);
    const totalCapacitySector = updatedParkings.reduce((sum, p) => sum + p.totalSpaces, 0);

    return {
      ...sector,
      availability: totalCapacitySector > 0 ? Math.floor((totalAvailableSector / totalCapacitySector) * 100) : 0,
      parkings: updatedParkings
    };
  });
};

// Manejar el cambio de ubicación (simulado)
const handleLocationChange = () => {
  if (isInsideCampus.value) {
    locationText.value = 'Usted se encuentra DENTRO del perímetro del campus.';
    updateSectorAvailability(); // Actualiza la disponibilidad al entrar al campus

    if (!updateInterval) {
      updateInterval = setInterval(updateSectorAvailability, 15000);
    }
  } else {
    locationText.value = 'Usted se encuentra FUERA del perímetro del campus. El mapa de estacionamientos no está disponible.';
    if (updateInterval) {
      clearInterval(updateInterval);
      updateInterval = null;
    }
  }
};

const simulateInside = () => {
  isInsideCampus.value = true;
  handleLocationChange();
};

const simulateOutside = () => {
  isInsideCampus.value = false;
  handleLocationChange();
};

onMounted(() => {
  parkingSectors.value = initialParkingSectors.map(sector => ({
    ...sector,
    availability: getRandomAvailability(),
  }));
  handleLocationChange();
});

const getAvailabilityColorClass = (availability) => {
  if (availability <= 30) {
    return 'bg-red-500'; // Muy ocupado
  } else if (availability <= 60) {
    return 'bg-orange-500'; // Moderado
  } else {
    return 'bg-green-500'; // Disponible
  }
};

// Computar el sector con mayor disponibilidad para notificaciones
const availableNotifications = computed(() => {
  if (parkingSectors.value.length === 0) {
    return [];
  }

  const sectorConMayorEspacio = parkingSectors.value.reduce((maxSector, currentSector) => {
    return (currentSector.availability > maxSector.availability) ? currentSector : maxSector;
  }, parkingSectors.value[0]);

  return [
    {
      name: sectorConMayorEspacio.name,
      availability: sectorConMayorEspacio.availability,
      message: sectorConMayorEspacio.availability > 30 ? '¡Este es tu mejor opción!' : 'Este es el que más tiene, pero igual está complicado.',
    }
  ];
});

watch(availableNotifications, (newNotifications) => {
  // Solo mostrar la notificación si hay contenido para mostrar
  if (newNotifications.length > 0) {
    showNotification.value = true;
    // Limpiar cualquier temporizador anterior para reiniciar la cuenta regresiva de 5 segundos
    if (notificationTimer) {
      clearTimeout(notificationTimer);
    }
    notificationTimer = setTimeout(() => {
      showNotification.value = false;
      notificationTimer = null;
    }, 5000);
  } else {
    // Si no hay sectores disponibles para notificar, asegurar que la notificación esté oculta
    showNotification.value = false;
    if (notificationTimer) {
      clearTimeout(notificationTimer);
      notificationTimer = null;
    }
  }
}, { deep: true, immediate: true }); // 'deep' para observar cambios dentro del array, 'immediate' para ejecutar al montar

// Estacionamientos del sector
const goToSectorDetail = (sectorId) => {
  const sector = parkingSectors.value.find(s => s.id === sectorId);
  if (sector) {
    selectedSector.value = sector;
    currentView.value = 'sectorDetail';
    sectorToHighlight.value = sector.coords;
    selectedParking.value = null;
    selectedSpaceId.value = null;
    destinationCoords.value = null;
  }
};

// Para volver a la vista general del campus y limpiar los campos
const goToCampusView = () => {
  currentView.value = 'campus';
  selectedSector.value = null;
  selectedParking.value = null;
  sectorToHighlight.value = null;
  selectedSpaceId.value = null;
};

const goToParkingDetail = (parkingId) => {
  if (selectedSector.value && selectedSector.value.parkings) {
    const parking = selectedSector.value.parkings.find(p => p.id === parkingId);
    if (parking) {
      selectedParking.value = parking;
      currentView.value = 'parkingDetail';
      selectedSpaceId.value = null;
      destinationCoords.value = null;
    }
  }
};

const backToSectorDetail = () => {
  currentView.value = 'sectorDetail';
  selectedParking.value = null;
  selectedSpaceId.value = null;
};

const selectSpace = (space) => {
  if (!space.isOccupied) { // Solo se puede seleccionar si no está ocupado
    if (selectedSpaceId.value === space.id) {
      selectedSpaceId.value = null; // Deseleccionar si ya estaba seleccionado
    } else {
      selectedSpaceId.value = space.id; // Seleccionar este espacio
    }
  } else {
    // Opcional: Mostrar un mensaje de que el espacio está ocupado
    console.log(`El espacio ${space.id} está ocupado.`);
    selectedSpaceId.value = null; // Asegurarse de que no haya un espacio ocupado seleccionado
  }
};

const handleContinue = () => {
  if (selectedSpaceId.value && selectedParking.value) {
    // Al presionar continuar, vamos a la vista de navegación
    // Las coordenadas del destino serán las del parking seleccionado
    destinationCoords.value = selectedParking.value.destinationCoords;
    currentView.value = 'navigation';
    // Opcional: aquí podrías marcar el espacio como 'ocupado' si fuera una reserva real
    // const selectedParkingIndex = selectedSector.value.parkings.findIndex(p => p.id === selectedParking.value.id);
    // const selectedSpaceIndex = selectedParking.value.spaces.findIndex(s => s.id === selectedSpaceId.value);
    // if (selectedParkingIndex !== -1 && selectedSpaceIndex !== -1) {
    //   selectedSector.value.parkings[selectedParkingIndex].spaces[selectedSpaceIndex].isOccupOccupied = true;
    //   // Y recalcular la disponibilidad, etc. Esto sería más complejo para una demo.
    // }

  } else {
    alert('Por favor, selecciona un espacio disponible para continuar.');
  }
};

const backToParkingDetail = () => {
  currentView.value = 'parkingDetail';
  // selectedParking ya debería estar establecido
  selectedSpaceId.value = null; // Opcional: Deseleccionar el espacio al volver
  destinationCoords.value = null; // Limpiar coords de destino
};

</script>