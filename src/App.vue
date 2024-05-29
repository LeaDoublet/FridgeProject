<template>
  <v-app>
    <v-main>
      <v-container fluid class="d-flex align-center justify-center text-center" style="height: 100vh;">
        <v-row>
          <v-col>
            <h1>Mon Frigo</h1>
            <FridgeList :refreshKey="key" />
            <AddProduct @productAdded="handleProductAdded" />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue';
import FridgeList from './components/FridgeList.vue';
import AddProduct from './components/AddProduct.vue';

const key = ref(0);

const handleProductAdded = () => {
  key.value += 1; // Recharge FridgeList en changeant la clé
};
</script>
