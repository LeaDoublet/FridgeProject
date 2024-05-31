<template>
  <v-app>
    <v-main>
      <v-container fluid class="d-flex align-center justify-center text-center" style="height: 100vh;">
        <v-row>
          <v-col>
            <h1>Mon Frigo</h1>
            <FridgeList :refreshKey="valuechange" :filteredProducts="filteredProducts" />
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
import SearchBar from './components/SearchBar.vue';
const valuechange = ref(0);
const filteredProducts = ref([]);


const handleProductAdded = () => {
  valuechange.value += 1; // Recharge FridgeList en changeant la clé
  console.log("je passe par handleproduct adeed" + valuechange.value)
};
</script>
