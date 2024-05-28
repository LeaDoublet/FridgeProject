<template>
    <v-list>
        <v-list-item v-for="product in products" :key="product.id">
            <v-list-item-content>
                <v-list-item-title>{{ product.nom }}</v-list-item-title>
                <v-list-item-subtitle>Quantité: {{ product.qte }}</v-list-item-subtitle>
            </v-list-item-content>
        </v-list-item>
    </v-list>
</template>

<script setup>

import { ref, onMounted } from 'vue';
const products = ref([]);

onMounted(async () => {
    try {
        const response = await fetch('https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits');
        if (!response.ok) {
            throw new Error('Erreur lors de la récupération des données');
        }
        const data = await response.json();
        products.value = data;
    } catch (error) {
        console.error('Erreur :', error.message);
    }
});
</script>
