<script setup>
import { ref, onMounted } from "vue";
import Navbar from "@/components/Navbar.vue";

const capturedPokemon = ref([]);
const showConfirmDialog = ref(false); // Menampilkan dialog konfirmasi
const selectedPokemon = ref(null); // Menyimpan Pokémon yang akan dihapus

onMounted(() => {
  capturedPokemon.value = JSON.parse(localStorage.getItem("pokemonList")) || [];
});

// Fungsi untuk menghapus Pokémon yang dipilih
const deletePokemon = () => {
  if (!selectedPokemon.value) return;

  // Hanya hapus Pokémon dengan uniqueId yang dipilih
  capturedPokemon.value = capturedPokemon.value.filter(pokemon => pokemon.uniqueId !== selectedPokemon.value.uniqueId);

  // Simpan kembali ke localStorage
  localStorage.setItem("pokemonList", JSON.stringify(capturedPokemon.value));

  // Tutup pop-up konfirmasi
  showConfirmDialog.value = false;
  selectedPokemon.value = null;
};

// Menampilkan pop-up konfirmasi
const confirmDelete = (pokemon) => {
  selectedPokemon.value = pokemon;
  showConfirmDialog.value = true;
};
</script>

<template>
  <Navbar />
  <div class="p-6">
    <h1 class="text-3xl font-bold text-center text-white">Captured Pokémon</h1>

    <!-- Jika belum ada Pokémon yang tertangkap -->
    <div v-if="capturedPokemon.length === 0" class="mt-4 text-gray-500 text-center">
      kamu belum menagkap pokemon
    </div>

    <!-- Grid untuk menampilkan Pokémon yang tertangkap -->
    <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 mt-6">
      <div 
        v-for="pokemon in capturedPokemon" 
        :key="pokemon.uniqueId" 
        class="bg-gray-800 p-4 rounded-lg shadow-md text-center relative"
      >
        <img :src="pokemon.image" :alt="pokemon.name" class="w-32 h-32 mx-auto" />
        <h2 class="text-xl font-semibold mt-2 capitalize text-white">{{ pokemon.name }}</h2>
        <p class="text-gray-400">Type: {{ pokemon.types.join(", ") }}</p>

        <!-- Tombol Delete -->
        <button 
          @click="confirmDelete(pokemon)" 
          class="absolute top-2 right-2 text-white px-2 py-1 rounded"
        >
          ❌
        </button>
      </div>
    </div>
  </div>

  <!-- Dialog Konfirmasi -->
  <div v-if="showConfirmDialog" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 animate-fadeIn">
    <div class="bg-white p-6 rounded-lg shadow-lg text-center w-96 relative">
      <h2 class="text-xl font-semibold text-red-500 mb-10">Lepaskan</h2>
      <p class="text-gray-600 mt-2 mb-10">Kamu yakin mau melepaskan <strong>{{ selectedPokemon?.name }}</strong>?</p>

      <div class="flex justify-center mt-4 gap-4">
        <button 
          @click="deletePokemon" 
          class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
        >
          Lepaskan
        </button>
        <button 
          @click="showConfirmDialog = false" 
          class="px-4 py-2 bg-green-300 rounded hover:bg-green-400 text-white"
        >
          Tidak
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Animasi fade-in */
@keyframes fadeIn {
  from { opacity: 0; transform: scale(0.9); }
  to { opacity: 1; transform: scale(1); }
}
.animate-fadeIn {
  animation: fadeIn 0.3s ease-in-out;
}
</style>
