<template>
    <div>
        <v-alert v-if="lowStockProducts.length" type="warning" dismissible class="custom-alert">
            <div v-for="product in lowStockProducts" :key="product.id">
                Le produit {{ product.nom }} est presque épuisé ({{ product.qte }} restant).
            </div>
        </v-alert>
        <SearchBar @search="filterProducts" />
        <v-table>
            <thead>
                <tr>
                    <th>Nom</th>
                    <th>Quantité</th>
                    <th>Actions</th>
                    <th>Images</th>
                </tr>
            </thead>
            <tbody>

                <tr v-for="product in paginatedProducts" :key="product.id">
                    <td>{{ product.nom }}</td>
                    <td>{{ product.qte }}</td>
                    <v-btn @click="addQuantity(product)" icon>
                        <v-icon>mdi-plus</v-icon>
                    </v-btn>
                    <v-btn @click="removeQuantity(product)" icon>
                        <v-icon>mdi-minus</v-icon>
                    </v-btn>
                    <td>
                        <v-img :src="product.photo" alt="" witdh="20" height="20"></v-img>
                    </td>
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
import SearchBar from './SearchBar.vue';

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
const lowStockThreshold = 5; // Ajout d'un stock minimum de produit à 5

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
        filterProducts(searchTerm.value); // Refiltrage des produits si besoin
    } catch (error) {
        console.error('Erreur :', error.message);
    }

}

const lowStockProducts = computed(() => {
    return products.value.filter(product => product.qte < lowStockThreshold);
});
onMounted(fetchProducts);

const filterProducts = (term) => {
    searchTerm.value = term;
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
// Fonction pour ajouter 1 à la quantité d' un produit (ajout rapide )
const addQuantity = async (product) => {
    try {
        // Ajoute 1 à la quantité du produit
        product.qte += 1;
        console.log(product)
        // Envoie une requête POST pour mettre à jour la quantité
        const response = await fetch(`https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(product)
        });
        const data = await response.json();
        if (data.status === 1) {
            console.log('Quantité ajoutée avec succès !');
            fetchProducts(); // Rafraîchi la liste des produits
        } else {
            throw new Error('Erreur lors de l\'ajout de la quantité');
        }
    } catch (error) {
        console.error('Erreur :', error.message);
    }
};

// Fonction pour supprimer 1 à la quantité d' un produit (ajout rapide )
const removeQuantity = async (product) => {
    if (product.qte > 0) {
        try {
            // Ajoute 1 à la quantité du produit
            product.qte -= 1;
            console.log(product)
            // Envoie une requête POST pour mettre à jour la quantité
            const response = await fetch(`https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(product)
            });
            const data = await response.json();
            if (data.status === 1) {
                console.log('Quantité ajoutée avec succès !');
                fetchProducts(); // Rafraîchi la liste des produits
            } else {
                throw new Error('Erreur lors de l\'ajout de la quantité');
            }
        } catch (error) {
            console.error('Erreur :', error.message);
        }
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
<style scoped>
.v-alert {
    margin-bottom: 10px;
    height: 40px;
    font-size: 14px;
}
</style>