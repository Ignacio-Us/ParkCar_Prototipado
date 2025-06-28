<template>
    <LMap :zoom="zoom" :center="center" ref="mapRef">
        <LTileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
            attribution='&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'>
        </LTileLayer>
        <LMarker v-for="sector in filteredSectors" :key="sector.id" :lat-lng="sector.coords"
            :icon="getParkingIcon(sector.availability)" :ref="(el) => { if (el) markerRefs[sector.id] = el }">
            <LPopup>
                <div>
                    <h3 class="font-bold text-lg">{{ sector.name }}</h3>
                    <p>Disponibilidad: {{ sector.availability }}%</p>
                    <p v-if="sector.availability > 70">¡Hay mucho espacio!</p>
                    <p v-else-if="sector.availability >= 30">Espacio moderado, apúrate.</p>
                    <p v-else>Casi lleno, busca alternativas.</p>
                </div>
            </LPopup>
        </LMarker>
    </LMap>
</template>

<script setup>
import { ref, watch, computed } from 'vue';
import { LMap, LTileLayer, LMarker, LPopup, LIcon } from '@vue-leaflet/vue-leaflet';
import L from 'leaflet'; // Necesario para crear íconos

L.Icon.Default.mergeOptions({
    iconRetinaUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon-2x.png',
    iconUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon.png',
    shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png',
});

const props = defineProps({
    sectors: {
        type: Array,
        required: true,
    },
    sectorToHighlight: {
        type: Array,
        default: null,
    },
    //Para filtrar los marcadores por un sector específico
    filterSectorId: {
        type: String,
        default: null,
    },
});

const zoom = ref(15);
const center = ref([-38.74802, -72.61684]); // Centro del campus

// Para acceder a las instancias del mapa y los marcadores
const mapRef = ref(null);
const markerRefs = ref({});

// Función para generar un icono de marcador personalizado según la disponibilidad
const getParkingIcon = (availability) => {
    let iconColor = 'green';
    if (availability <= 30) {
        iconColor = 'red';
    } else if (availability <= 60) {
        iconColor = 'orange';
    }

    const svgIcon = `
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="${iconColor}" stroke="#ffffff" stroke-width="0.5" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10" fill="${iconColor}" stroke="#fff" stroke-width="1.5"/>
          <path d="M8 7 H16 M8 12 H16 M8 17 H16 M8 7 V17" stroke="#fff" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      `;

    return L.icon({
        iconUrl: `data:image/svg+xml;base64,${btoa(svgIcon)}`,
        iconSize: [22, 23],
        iconAnchor: [12, 20],
        popupAnchor: [0, -28],
    });
};

// Para filtrar los sectores que se muestran en el mapa
const filteredSectors = computed(() => {
    if (props.filterSectorId) {
        return props.sectors.filter(sector => sector.id === props.filterSectorId);
    }
    return props.sectors;
});

// Para reaccionar cuando se selecciona un sector desde App.vue
watch(() => props.sectorToHighlight, (newCoords) => {
    if (newCoords && mapRef.value) {
        const targetSector = props.sectors.find(s =>
            s.coords[0] === newCoords[0] && s.coords[1] === newCoords[1]
        );

        // Solo abre el popup si el sector está en los 'filteredSectors'
        const isTargetSectorVisible = filteredSectors.value.some(s => s.id === targetSector.id);

        if (targetSector && markerRefs.value[targetSector.id] && isTargetSectorVisible) {
            const marker = markerRefs.value[targetSector.id].leafletObject;
            marker.openPopup();
            mapRef.value.leafletObject.setView(newCoords, zoom.value);
        } else if (mapRef.value.leafletObject && !isTargetSectorVisible) {
            mapRef.value.leafletObject.setView(newCoords, zoom.value);
        }
    }
}, { immediate: true }); // Tener immediate: true para que se active al cargar si ya hay un sector seleccionado.
</script>