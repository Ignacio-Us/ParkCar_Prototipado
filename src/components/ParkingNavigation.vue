<template>
  <div class="navigation-container">
    <div id="map-navigation" ref="mapContainer" class="h-full w-full rounded-lg overflow-hidden"></div>
    <div class="routing-instructions bg-white p-4 rounded-lg shadow-lg mt-4 max-h-64 overflow-y-auto">
      <h3 class="text-xl font-bold mb-2">Indicaciones:</h3>
      <div v-if="instructions.length === 0" class="text-gray-600">Calculando ruta...</div>
      <ol class="list-decimal pl-5">
        <li v-for="(instruction, index) in instructions" :key="index" class="mb-1 text-gray-800">
          {{ instruction.text }} ({{ (instruction.distance / 1000).toFixed(2) }} km)
        </li>
      </ol>
    </div>
    <button @click="$emit('back')" class="mt-4 px-6 py-3 bg-blue-500 hover:bg-blue-600 text-white font-semibold rounded-lg shadow-md transition-colors">
      ← Volver al Estacionamiento
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import L from 'leaflet'; // Importa el objeto L de Leaflet
import 'leaflet-routing-machine'; // Importa el JS del plugin
import 'leaflet-routing-machine/dist/leaflet-routing-machine.css'; // Importa el CSS del plugin
import 'leaflet-control-geocoder/dist/Control.Geocoder.css'; // Si usas el geocodificador de L.Control.Geocoder.nominatim()
import 'leaflet-control-geocoder'; // Esto suele ser suficiente para importar el JS

const props = defineProps({
  destinationCoords: {
    type: Array, // [lat, lng]
    required: true
  }
});

const emit = defineEmits(['back']); // Evento para emitir cuando se quiera volver

const mapContainer = ref(null);
let mapInstance = null;
let userMarker = null;
let routingControl = null;
let watchId = null; // Para el ID del watcher de geolocalización
const instructions = ref([]);

onMounted(() => {
  // Asegúrate de que el contenedor del mapa exista antes de inicializar Leaflet
  if (!mapContainer.value) {
    console.error("Contenedor del mapa no encontrado.");
    return;
  }

  mapInstance = L.map(mapContainer.value).setView(props.destinationCoords, 16); // Centrar en el destino

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors'
  }).addTo(mapInstance);

  // Define las coordenadas fijas para el usuario
  const fixedUserLat = -38.74903151261987; // Latitud de ejemplo (cerca del centro de Temuco)
  const fixedUserLng = -72.61746887521349; // Longitud de ejemplo (cerca del centro de Temuco)
  const fixedUserLatLng = [fixedUserLat, fixedUserLng];

  // Puedes añadir una bandera para alternar entre geolocalización real y coordenadas fijas
  const useFixedLocation = true; // Cambia a 'false' si quieres usar la geolocalización real

  if (useFixedLocation) {
    // Usar las coordenadas fijas
    const userLatLng = fixedUserLatLng;

    if (userMarker) {
      userMarker.setLatLng(userLatLng);
    } else {
      userMarker = L.marker(userLatLng).addTo(mapInstance)
        .bindPopup("Tu ubicación simulada").openPopup(); // Cambia el texto del popup
    }

    if (!routingControl) {
      initRouting(userLatLng, props.destinationCoords);
    } else {
      routingControl.spliceWaypoints(0, 1, userLatLng);
    }

    mapInstance.panTo(userLatLng); // Centrar el mapa en la ubicación simulada
  } else if (navigator.geolocation) {
    // Si no usamos la ubicación fija, y el navegador soporta geolocalización, la usamos
    watchId = navigator.geolocation.watchPosition(
      (position) => {
        const lat = position.coords.latitude;
        const lng = position.coords.longitude;
        const userLatLng = [lat, lng];

        if (userMarker) {
          userMarker.setLatLng(userLatLng);
        } else {
          userMarker = L.marker(userLatLng).addTo(mapInstance)
            .bindPopup("Tu ubicación").openPopup();
        }

        if (!routingControl) {
          initRouting(userLatLng, props.destinationCoords);
        } else {
          routingControl.spliceWaypoints(0, 1, userLatLng);
        }

        mapInstance.panTo(userLatLng);
      },
      (error) => {
        console.error("Error obteniendo la ubicación: " + error.message);
        alert("No se pudo obtener tu ubicación actual. Usando ubicación predeterminada para el origen de la ruta.");
        if (!routingControl) {
          initRouting(mapInstance.getCenter(), props.destinationCoords);
        }
      },
      {
        enableHighAccuracy: true,
        maximumAge: 10000,
        timeout: 5000
      }
    );
  } else {
    // Si no hay geolocalización y no se usa la ubicación fija, solo alerta
    alert("Tu navegador no soporta la Geolocalización. La navegación en tiempo real no estará disponible.");
    // Si no hay geolocalización, inicializa la ruta con un origen predeterminado (ej. el centro del mapa)
    initRouting(mapInstance.getCenter(), props.destinationCoords);
  }
});


onUnmounted(() => {
  // Limpiar el watcher de geolocalización al desmontar el componente
  if (watchId) {
    navigator.geolocation.clearWatch(watchId);
  }
  // Limpiar el control de ruteo y el mapa
  if (routingControl) {
    mapInstance.removeControl(routingControl);
    routingControl = null;
  }
  if (mapInstance) {
    mapInstance.remove();
    mapInstance = null;
  }
});

function initRouting(origin, destination) {
  // Asegúrate de que L.Control.Geocoder esté disponible si lo usas
  // Si no lo necesitas, puedes quitarlo
  const geocoder = L.Control.Geocoder.nominatim();

  routingControl = L.Routing.control({
    waypoints: [
      L.latLng(origin[0], origin[1]),
      L.latLng(destination[0], destination[1])
    ],
    router: L.Routing.osrmv1({
      serviceUrl: 'https://router.project-osrm.org/route/v1'
    }),
    fitSelectedRoutes: true,
    lineOptions: {
      styles: [{ color: '#6FA1EC', weight: 6, opacity: 0.8 }]
    },
    language: 'es',
    showAlternatives: false,
    geocoder: geocoder, // Integración del geocodificador
    // Para ocultar la UI por defecto de Leaflet Routing Machine y manejar las instrucciones nosotros
    // show: false,
    // waypointsCollapsed: true
  }).addTo(mapInstance);

  // Escuchar el evento 'routesfound' para obtener las instrucciones
  routingControl.on('routesfound', function(e) {
    const route = e.routes[0]; // Tomamos la primera ruta
    instructions.value = route.instructions.map(inst => ({
      text: inst.text,
      distance: inst.distance
    }));
    console.log("Ruta encontrada:", instructions.value);
  });

  // Opcional: Escuchar cuando la ruta falla
  routingControl.on('routingerror', function(e) {
      console.error("Error al calcular la ruta:", e);
      instructions.value = [{ text: "Error al calcular la ruta. Por favor, intenta de nuevo.", distance: 0 }];
  });
}
</script>

<style scoped>
.navigation-container {
  display: flex;
  flex-direction: column;
  height: 100%; /* Ajusta según sea necesario para que ocupe el espacio */
}

#map-navigation {
  flex-grow: 1; /* Ocupa el espacio disponible */
  min-height: 400px; /* Altura mínima para que el mapa sea visible */
  width: 100%;
}

.routing-instructions {
  flex-shrink: 0; /* No se encoge */
  /* Otros estilos ya definidos en el template */
}

/* Puedes añadir más estilos para los elementos de Leaflet Routing Machine si quieres personalizarlos */
.leaflet-routing-container {
  display: none; /* Oculta la UI por defecto si quieres manejar las instrucciones manualmente */
}
</style>