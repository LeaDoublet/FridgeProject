<template>
    <div>
        <h2>Liste des produits dans le frigo :</h2>
        <v-text-field v-model="searchTerm" label="Rechercher des produits" outlined @input="filterProducts" />
        <v-table>
            <thead>
                <tr>
                    <th>Nom</th>
                    <th>Quantité</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="product in paginatedProducts" :key="product.id">
                    <td>{{ product.nom }}</td>
                    <td>{{ product.qte }}</td>
                </tr>
            </tbody>
        </v-table>
        <v-btn @click="prevPage" :disabled="currentPage === 1">Précédent</v-btn>
        <v-btn @click="nextPage" :disabled="currentPage === totalPages">Suivant</v-btn>
        <p>Page {{ currentPage }} sur {{ totalPages }}</p>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
const products = ref([]);
const filteredProducts = ref([]);
const searchTerm = ref('');
const currentPage = ref(1);
const itemsPerPage = 10;
const refreshKey2 = defineProps(['refreshKey']);

//On recupère les produits de l'api avec le bon id etudiant
const fetchProducts = async () => {
    console.log('jappelle fetch product')
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

// Calcul des produits à afficher pour la page actuelle
const paginatedProducts = computed(() => {
    const start = (currentPage.value - 1) * itemsPerPage;
    const end = start + itemsPerPage;
    return filteredProducts.value.slice(start, end);
});

// Calcul du nombre total de pages
const totalPages = computed(() => {
    return Math.ceil(filteredProducts.value.length / itemsPerPage);
});

const nextPage = () => {
    if (currentPage.value < totalPages.value) {
        currentPage.value++;
    }
};

const prevPage = () => {
    if (currentPage.value > 1) {
        currentPage.value--;
    }
};

// Surveille les changements dans refreshKey et MAJ liste produits
watch(refreshKey2, () => {
    console.log("je passe par le rafraichissement de produits")
    fetchProducts();
});
watch(searchTerm, filterProducts);
</script>
