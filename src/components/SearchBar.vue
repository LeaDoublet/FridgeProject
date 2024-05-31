<template>
    <div>
        <v-text-field v-model="searchTerm" label="Rechercher des produits" @input="searchProducts" />
    </div>
</template>

<script setup>
import { ref } from 'vue';
const searchTerm = ref('');
const emit = defineEmits(['search']);
const searchProducts = () => {
    // On créer un événement personnalisé avec emit -->https://vuejs.org/guide/components/events.html
    emit('search', searchTerm.value);
}
</script>
