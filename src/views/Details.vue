<script setup>
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import Navbar from "@/components/Navbar.vue";

const route = useRoute(); 
const pokemon = ref(null);
const isLoading = ref(true);
const errorMessage = ref("");
const activeTab = ref("about");
const isCatching = ref(false);
const caughtMessage = ref(null);
let capturedPokemon = JSON.parse(localStorage.getItem("pokemonList")) || [];

const fetchPokemonDetail = async () => {
  try {
    const res = await fetch(
      `https://pokeapi.co/api/v2/pokemon/${route.params.id}`
    );
    if (!res.ok) throw new Error("Gagal fetching data");
    const data = await res.json();

    pokemon.value = {
      id: data.id,
      name: data.name.charAt(0).toUpperCase() + data.name.slice(1),
      image: data.sprites.other["official-artwork"].front_default,
      types: data.types.map((t) => t.type.name), 
      height: `${data.height / 10} m`,
      weight: `${data.weight / 10} kg`,
      abilities: data.abilities.map((a) => a.ability.name), 
      experience: data.base_experience,
    };
  } catch (error) {
    errorMessage.value = "Error fetching Pokémon data.";
  } finally {
    isLoading.value = false;
  }
};

const catchPokemon = () => {
  if (!pokemon.value || isCatching.value) return;
    
  isCatching.value = true; // Mulai animasi menangkap
  caughtMessage.value = null; // Reset pesan sebelumnya
    
  setTimeout(() => {
    const success = Math.random() > 0.5; // 50% peluang menangkap
    if (success) {
      const newPokemon = {
        id: pokemon.value.id,
        name: pokemon.value.name,
        types: pokemon.value.types,
        image: pokemon.value.image,
        uniqueId: Date.now(),
      };
      capturedPokemon.push(newPokemon);
      localStorage.setItem("pokemonList", JSON.stringify(capturedPokemon));

      // Tampilkan pesan sukses dengan kartu Pokémon
      caughtMessage.value = {
        success: true,
        pokemon: newPokemon
      };
    } else {
      // Tampilkan pesan gagal
      caughtMessage.value = {
        success: false
      };
    }
    isCatching.value = false; // Hentikan animasi menangkap
  }, 2000);
};

// Menutup pesan tangkapan
const closeMessage = () => {
  caughtMessage.value = null;
};

onMounted(fetchPokemonDetail);
</script>

<template>
  <Navbar />
  <div class="bg-black min-h-screen flex items-center justify-center p-4">
    <div v-if="isLoading" class="text-center text-lg font-semibold text-white">
      Loading...
    </div>
    <div v-else-if="errorMessage" class="text-center text-red-500 font-semibold">
      {{ errorMessage }}
    </div>

    <div v-else class="bg-gray-800 rounded-2xl shadow-lg p-6 max-w-3xl w-full flex flex-col md:flex-row">
      <div class="flex flex-col items-center text-center md:w-1/2">
        <img :src="pokemon.image" :alt="pokemon.name" class="w-64 h-64" />
        <h1 class="text-3xl font-bold text-gray-500 mt-4">
          #{{ pokemon.id }} {{ pokemon.name }}
        </h1>

        <!-- Tombol Menangkap Pokémon -->
       <!-- Tombol Menangkap Pokémon -->
<button
  @click="catchPokemon"
  class="mt-4 px-6 py-2 rounded bg-red-500 text-white font-bold transition-all duration-300 hover:bg-red-600  active:scale-95 flex items-center justify-center"
  :disabled="isCatching"
>
  <img src="/icon.png" class="animate-bounce w-8 h-8" alt="Pokeball" />
  <span v-if="!isCatching" class="mr-1 ml-1 font-bold">Catch {{ pokemon.name }}</span>
  <span v-else class="animate-bounce">Throwing Pokéball...</span>
</button>
      </div>

      <!-- Detail Pokémon -->
      <div class="md:w-1/2 p-4">
        <div class="flex gap-2">
          <button
            @click="activeTab = 'about'"
            :class="activeTab === 'about' ? 'bg-yellow-400' : 'bg-gray-200'"
            class="px-3 py-1 rounded-md text-gray-900 font-semibold"
          >
            About
          </button>
          <button
            @click="activeTab = 'stats'"
            :class="activeTab === 'stats' ? 'bg-yellow-400' : 'bg-gray-200'"
            class="px-3 py-1 rounded-md text-gray-800"
          >
            Stats
          </button>
          <button
            @click="activeTab = 'moves'"
            :class="activeTab === 'moves' ? 'bg-yellow-400' : 'bg-gray-200'"
            class="px-3 py-1 rounded-md text-gray-800"
          >
            Moves
          </button>
        </div>

        <div v-if="activeTab === 'about'" class="mt-8 space-y-4">
          <p class="text-gray-400">
            <strong>Types:</strong>
            <span
              v-for="type in pokemon.types"
              :key="type"
              class="px-2 py-1 bg-sky-400 text-slate-800 rounded-full mx-1 text-sm capitalize"
            >
              {{ type }}
            </span>
          </p>
          <p class="text-gray-400"><strong>Height:</strong> {{ pokemon.height }}</p>
          <p class="text-gray-400"><strong>Weight:</strong> {{ pokemon.weight }}</p>
          <p class="text-gray-400">
            <strong>Abilities:</strong>
            <span
              v-for="ability in pokemon.abilities"
              :key="ability"
              class="px-2 py-1 bg-yellow-400 text-slate-800 rounded-full mx-1 text-sm capitalize"
            >
              {{ ability }}
            </span>
          </p>
          <p class="text-gray-400"><strong>Experience:</strong> {{ pokemon.experience }} Exp</p>
        </div>
        
      </div>
    </div>
  </div>



  <!-- Pesan tangkapan Pokémon -->
  <div v-if="caughtMessage" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 animate-fadeIn">
    <div class="bg-white p-6 rounded-lg shadow-lg text-center w-96 relative">
      <button @click="closeMessage" class="absolute top-2 right-2 text-gray-500 hover:text-gray-700">✖</button>

      <div v-if="caughtMessage.success" class="animate-fadeIn">
        <h2 class="text-xl font-semibold text-green-700">horeee dapet {{ caughtMessage.pokemon.name }}!</h2>
        <img :src="caughtMessage.pokemon.image" alt="Pokemon" class="w-32 mx-auto my-2" />
        <p class="text-gray-600">Type: {{ caughtMessage.pokemon.types.join(", ") }}</p>
      </div>
      <div v-else class="animate-fadeIn">
        <h2 class="text-xl font-semibold text-red-700">Pokemon Mu kabur......</h2>
      </div>

      <button @click="closeMessage" class="mt-4 px-4 py-2 bg-red-500 rounded hover:bg-red-600 text-white">
        Close
      </button>
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
