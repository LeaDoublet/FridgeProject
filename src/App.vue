<template>
  <v-app>
    <v-main>
      <v-container fluid class="d-flex align-center justify-center text-center" style="height: 100vh;">
        <v-row>
          <v-col>
            <h1>Mon Frigo</h1>
            <FridgeList :products="items" />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue';
import FridgeList from './components/FridgeList.vue';

const items = ref([
  { id: 1, nom: 'Lait', qte: 2 },
  { id: 2, nom: 'Beurre', qte: 1 },
  { id: 3, nom: 'Fromage', qte: 5 }
]);
</script>
