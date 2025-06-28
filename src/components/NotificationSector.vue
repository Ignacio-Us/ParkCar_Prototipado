<template>
    <ul v-if="notifications.length > 0" class="list-disc list-inside space-y-2 text-gray-800">
        <li v-for="sector in notifications" :key="sector.name">
            El <span class="font-semibold">{{ sector.name }}</span> tiene un {{ sector.availability }}% de espacios
            disponibles.
            <span v-if="sector.message" class="italic text-gray-600">({{ sector.message }})</span>
        </li>
    </ul>
    <p v-else class="text-gray-500 text-center">
        En este momento no hay información de estacionamientos disponible o no se encontraron sectores.
    </p>
</template>

<script setup>
import { defineProps } from 'vue';

const props = defineProps({
    notifications: {
        type: Array,
        required: true,
    },
});
</script>