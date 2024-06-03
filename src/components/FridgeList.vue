<template>
    <div>
        <h2>Liste des produits dans le frigo :</h2>
        <v-text-field v-model="searchTerm" label="Rechercher des produits" outlined @input="filterProducts" />
        <v-table>
            <thead>
                <tr>
                    <th>Nom</th>
                    <th>Quantité</th>
                    <!--<th>Image</th>-->
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="product in paginatedProducts" :key="product.id">
                    <td>{{ product.nom }}</td>
                    <td>{{ product.qte }}</td>
                    <!--<td><img :src="product.image" alt="Image produit" width="50" height="50"></td>-->
                    <td>
                        <v-btn @click="openEditDialog(product)" color="pink">Modifier</v-btn>
                        <v-btn @click="confirmDelete(product.id)" color="purple">Supprimer</v-btn>
                        <v-btn @click="viewInfos(product.id)" color="white"> Infos</v-btn>
                    </td>
                </tr>
            </tbody>
        </v-table>
        <v-btn @click="prevPage" :disabled="currentPage === 1">Précédent</v-btn>
        <v-btn @click="nextPage" :disabled="currentPage === totalPages">Suivant</v-btn>
        <p>Page {{ currentPage }} sur {{ totalPages }}</p>
        <EditProductDialog :product="selectedProduct" :showDialog="editDialog" @update:showDialog="editDialog = $event"
            @close="closeEditDialog" @save="updateProduct" />
        <InfosDialog :product="selectedProductInfo" :showDialog="infoDialog" @update:showDialog="infoDialog = $event" />
    </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import EditProductDialog from './EditProductDialog.vue';
import InfosDialog from './InfoProductDialog.vue';

const products = ref([]);
const filteredProducts = ref([]);
const searchTerm = ref('');
const currentPage = ref(1);
const itemsPerPage = 5;
const refreshKey2 = defineProps(['refreshKey']);
const editDialog = ref(false);
const infoDialog = ref(false);
const selectedProduct = ref({});
const selectedProductInfo = ref(null);

// On récupère les produits de l'API avec le bon ID étudiant
const fetchProducts = async () => {
    console.log('jappelle fetch product');
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

const confirmDelete = (productId) => {
    if (confirm("Êtes-vous sûr de vouloir supprimer ce produit ?")) {
        deleteProduct(productId);
    }
}

const deleteProduct = async (productId) => {
    console.log('on supprime le produit avec l id' + productId);
    try {
        const response = await fetch(`https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits/${productId}`, {
            method: 'DELETE'
        });
        const result = await response.json();
        if (result.status === 1) {
            fetchProducts();
        } else {
            throw new Error('Erreur lors de la suppression du produit');
        }
    } catch (error) {
        console.error('Erreur :', error.message);
    }
}

const openEditDialog = (product) => {
    selectedProduct.value = { ...product };
    editDialog.value = true;
}

const closeEditDialog = () => {
    editDialog.value = false;
}

// On appelle l'api en PUT et on lui donne le produit dans le body pour pouvoir modifier.
const updateProduct = async (product) => {
    try {
        const response = await fetch(`https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(product)
        });
        const result = await response.json();
        if (result.status === 1) {
            fetchProducts();
            closeEditDialog();
        } else {
            throw new Error('Erreur lors de la mise à jour du produit');
        }
    } catch (error) {
        console.error('Erreur :', error.message);
    }
}

const viewInfos = async (productId) => {
    try {
        const response = await fetch(`https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits/${productId}`);
        if (!response.ok) {
            throw new Error('Erreur lors de la récupération des informations du produit');
        }
        const data = await response.json();
        if (data.status === 0) {
            throw new Error('Produit non trouvé');
        }
        selectedProductInfo.value = data;
        infoDialog.value = true;
    } catch (error) {
        console.error('Erreur :', error.message);
    }
};

// Surveille les changements dans refreshKey et MAJ liste produits
watch(refreshKey2, () => {
    console.log("je passe par le rafraichissement de produits");
    fetchProducts();
});
watch(searchTerm, filterProducts);
</script>
