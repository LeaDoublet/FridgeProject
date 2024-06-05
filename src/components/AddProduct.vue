<template>
    <div>
        <h4>Ajouter un nouveau produit :</h4>
        <v-form @submit.prevent="addProduct">
            <v-row>
                <v-col cols="6">
                    <v-text-field v-model="newProduct.nom" label="Nom du produit" required
                        class="form-field slim-field"></v-text-field>
                </v-col>
                <v-col cols="6">
                    <v-text-field v-model="newProduct.photo" label="Photo" class="form-field slim-field"></v-text-field>
                </v-col>
            </v-row>
            <v-text-field v-model="newProduct.qte" label="Quantité" required type="number"
                class="form-field slim-field"></v-text-field>
            <v-btn type="submit" color="purple" class="form-field slim-button">Ajouter</v-btn>
            <v-alert v-if="error" type="error" class="form-field">{{ error }}</v-alert>
        </v-form>
    </div>
</template>

<script setup>
import { ref, defineEmits } from 'vue';

const newProduct = ref({
    nom: '',
    qte: '',
    photo: ''
});

const error = ref('');
const emit = defineEmits(['productAdded']);


const addProduct = async () => {
    if (newProduct.value.qte < 0) {
        error.value = 'La quantité ne peut pas être négative';
        return;
    }
    error.value = ''; // Réinitialisation du message d'erreur

    console.log('Sending product data:', newProduct.value)
    try {
        const response = await fetch('https://webmmi.iut-tlse3.fr/~pecatte/frigo/public/4/produits', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(newProduct.value)
        });
        const data = await response.json();
        if (data.status === 1) {
            console.log('Produit ajouté avec succès !');
            // On emet un événement pour indiquer que le produit a été ajouté aux autres composants
            emit('productAdded');
            console.log('ca passe dessous le emit');
            // On réinitialise le formulaire après l'ajout réussi
            newProduct.value = { nom: '', qte: '', photo: '' };
        } else {
            console.error('Erreur lors de l ajout du produit');
        }
    } catch (error) {
        console.error('Erreur :', error.message);
    }
}
</script>

<style scoped>
.form-container {
    max-width: 400px;
    margin: 0 auto;
}

.form-field {
    width: 100%;
    margin: auto;
}
</style>