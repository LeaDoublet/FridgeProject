<template>
    <div>
        <v-text-field v-model="searchTerm" label="Rechercher des produits" @input="searchProducts" />
    </div>
</template>

<script setup>
import { ref } from 'vue';
const searchTerm = ref('');
const searchProducts = () => {
    // On créer un événement personnalisé avec emit -->https://vuejs.org/guide/components/events.html
    emit('search', searchTerm.value);
}
</script>
