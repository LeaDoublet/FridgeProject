<template>
    <div>
        <h2>Liste des produits dans le frigo :</h2>
        <v-text-field v-model="searchTerm" label="Rechercher des produits" outlined @input="filterProducts" />
        <v-list>
            <v-list-item v-for="product in filteredProducts" :key="product.id">
                <v-list-item-content>
                    <v-list-item-title>{{ product.nom }}</v-list-item-title>
                    <v-list-item-subtitle>Quantité: {{ product.qte }}</v-list-item-subtitle>
                </v-list-item-content>
            </v-list-item>
        </v-list>
    </div>
</template>

<script setup>

import { ref, onMounted, watch } from 'vue';
const products = ref([]);
const filteredProducts = ref([]);
const searchTerm = ref('');

//On recupère les produits de l'api avec le bon id etudiant
const fetchProducts = async () => {
    try {
        const response = await fetch('https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits');
        if (!response.ok) {
            throw new Error('Erreur lors de la récupération des données');
        }
        const data = await response.json();
        products.value = data;
        filterProducts(); // Refiltrage des produits si besoin
    } catch (error) {
        console.error('Erreur :', error.message);
    }
}

onMounted(fetchProducts);

const filterProducts = () => {
    if (!searchTerm.value) {
        // Si le champ de recherche est vide, on affiche tous les produits
        filteredProducts.value = products.value;
    } else {
        // Sinon on filtre les produits en fonction du terme de recherche
        filteredProducts.value = products.value.filter(product => product.nom.toLowerCase().includes(searchTerm.value.toLowerCase()));
    }
}


watch(products, filterProducts);


</script>
