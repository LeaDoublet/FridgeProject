<template>
    <v-dialog v-model="internalShowDialog" max-width="500px">
        <v-card>
            <v-card-title>
                <h3>Modifier le produit</h3>
            </v-card-title>
            <v-card-text>
                <v-container>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="internalProduct.nom" label="Nom du produit" outlined required />
                        </v-col>
                        <v-col cols="12">
                            <v-text-field v-model="internalProduct.qte" label="Quantité" type="number" outlined
                                required />
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="pink" text @click="close">Annuler</v-btn>
                <v-btn color="pink" text @click="save">Sauvegarder</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script setup>
import { ref, watch, defineProps, defineEmits } from 'vue';

const props = defineProps({
    product: Object,
    showDialog: Boolean,
});
const emit = defineEmits(['update:showDialog', 'close', 'save']);

const internalProduct = ref({ ...props.product });
const internalShowDialog = ref(props.showDialog);

watch(() => props.product, (newVal) => {
    internalProduct.value = { ...newVal };
});

watch(() => props.showDialog, (newVal) => {
    internalShowDialog.value = newVal;
});

watch(internalShowDialog, (newVal) => {
    emit('update:showDialog', newVal);
});

const close = () => {
    emit('close');
    internalShowDialog.value = false;
};

const save = () => {
    emit('save', internalProduct.value);
    internalShowDialog.value = false;
};
</script>

<style scoped>
.headline {
    font-weight: bold;
}
</style>