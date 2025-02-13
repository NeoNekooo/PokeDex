<script setup>
import { ref, onMounted, computed } from "vue";
import { useRoute } from "vue-router";
import icon from "@/components/icon.vue";
import icon_side from "@/components/icon_side.vue";

const route = useRoute();
const pokemon = ref(null);
const isLoading = ref(true);
const errorMessage = ref("");
const activeTab = ref("about");
const isCatching = ref(false);
const caughtMessage = ref(null);
let capturedPokemon = JSON.parse(localStorage.getItem("pokemonList")) || [];

const typeColors = {
  grass: "bg-green-500",
  fire: "bg-red-500",
  water: "bg-blue-500",
  electric: "bg-yellow-400",
  psychic: "bg-purple-500",
  ice: "bg-cyan-300",
  dragon: "bg-indigo-600",
  dark: "bg-gray-800",
  fairy: "bg-pink-300",
  normal: "bg-gray-400",
  fighting: "bg-orange-600",
  flying: "bg-sky-400",
  poison: "bg-purple-400",
  ground: "bg-yellow-700",
  rock: "bg-yellow-800",
  bug: "bg-lime-500",
  ghost: "bg-indigo-400",
  steel: "bg-gray-500",
};

const backgroundColor = computed(() => {
  if (pokemon.value && pokemon.value.types.length > 0) {
    const primaryType = pokemon.value.types[0];
    return typeColors[primaryType] || "bg-gray-200";
  }
  return "bg-gray-200";
});

const fetchPokemonDetail = async (id) => {
  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
    if (!res.ok) throw new Error("Gagal fetching data");
    const data = await res.json();
    const totalStats = data.stats.reduce(
      (sum, stat) => sum + stat.base_stat,
      0
    );
    const averageStats = totalStats / data.stats.length;

    pokemon.value = {
      id: data.id,
      name: data.name.charAt(0).toUpperCase() + data.name.slice(1),
      image:
        data.sprites.other["official-artwork"].front_default || "/default.png",
      types: data.types.map((t) => t.type.name),
      height: `${data.height / 10} (M)`,
      weight: `${data.weight / 10} (KG)`,
      abilities: data.abilities.map((a) => a.ability.name),
      experience: data.base_experience,
      moves: data.moves,
      stats: data.stats,
      averageStats: averageStats.toFixed(2),
    };
  } catch (error) {
    errorMessage.value = "Error fetching Pokémon data.";
  } finally {
    isLoading.value = false;
  }
};

const catchPokemon = () => {
  if (!pokemon.value || isCatching.value) return;

  isCatching.value = true;
  caughtMessage.value = null;

  setTimeout(() => {
    const success = Math.random() > 0.5;
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

      caughtMessage.value = {
        success: true,
        pokemon: newPokemon,
      };
    } else {
      caughtMessage.value = {
        success: false,
      };
    }
    isCatching.value = false;
  }, 2000);
};

const closeMessage = () => {
  caughtMessage.value = null;
};

const goToPreviousPokemon = () => {
  if (pokemon.value && pokemon.value.id > 1) {
    fetchPokemonDetail(pokemon.value.id - 1);
  }
};

const goToNextPokemon = () => {
  if (pokemon.value) {
    fetchPokemonDetail(pokemon.value.id + 1);
  }
};

onMounted(() => {
  fetchPokemonDetail(route.params.id);
});
</script>

<template>
  <icon />
  <icon_side />
  <div
    :class="[
      'min-h-screen',
      'flex',
      'items-center',
      'justify-center',
      'p-4',
      backgroundColor,
    ]"
  >
    <div v-if="isLoading" class="text-center text-lg font-semibold text-white">
      Loading...
    </div>
    <div
      v-else-if="errorMessage"
      class="text-center text-red-500 font-semibold"
    >
      {{ errorMessage }}
    </div>

    <div v-else class="w-full max-w-3xl">
      <div class="rounded-2xl p-6 w-full flex flex-col md:flex-row bg-transparent">
        <div class="desktop-buttons">
          <button
            @click="goToPreviousPokemon"
            class="btn btn-neutral size-10 rounded-full m-auto mr-5 pl-4 pr-3"
          >
            <
          </button>
        </div>

        <div
          class="md:w-1/2 flex flex-col items-center bg-transparent outline mr-3 p-4 rounded-2xl"
        >
          <h1 class="text-xl font-bold text-white">#{{ pokemon.id }}</h1>
          <img :src="pokemon.image" :alt="pokemon.name" class="w-64 h-64" />
          <h1 class="text-3xl font-bold text-white mt-4">
            {{ pokemon.name }}
          </h1>

          <button
            @click="catchPokemon"
            class="mt-4 px-6 py-2 rounded bg-red-500 text-white font-bold transition-all duration-300 hover:bg-red-600 outline active:scale-95 flex items-center justify-center"
            :disabled="isCatching"
          >
            <img
              src="../assets/pokeballs.png"
              class="animate-bounce w-8 h-8"
              alt="Pokeball"
            />
            <span v-if="!isCatching" class="mr-1 ml-1 font-bold"
              >Catch {{ pokemon.name }}</span
            >
            <span v-else class="animate-bounce">Throwing Pokéball...</span>
          </button>
        </div>

        <div class="md:w-1/2 bg-gray-800 rounded-2xl shadow-lg p-6 max-w-lg">
          <div class="flex gap-2">
            <button
              @click="activeTab = 'about'"
              :class="activeTab === 'about' ? 'bg-cyan-400' : 'bg-gray-200'"
              class="px-3 py-1 rounded-md text-gray-900 font-semibold"
            >
              About
            </button>
            <button
              @click="activeTab = 'stats'"
              :class="activeTab === 'stats' ? 'bg-cyan-400' : 'bg-gray-200'"
              class="px-3 py-1 rounded-md text-gray-800"
            >
              Stats
            </button>
            <button
              @click="activeTab = 'moves'"
              :class="activeTab === 'moves' ? 'bg-cyan-400' : 'bg-gray-200'"
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
                class="px-2 py-1 bg-yellow-400 text-slate-800 rounded mx-1 text-sm capitalize"
              >
                {{ type }}
              </span>
            </p>
            <p class="text-gray-400">
              <strong>Height:</strong> {{ pokemon.height }}
            </p>
            <p class="text-gray-400">
              <strong>Weight:</strong> {{ pokemon.weight }}
            </p>
            <p class="text-gray-400">
              <strong>Abilities:</strong>
              <span
                v-for="ability in pokemon.abilities"
                :key="ability"
                class="px-2 py-1 bg-yellow-400 mt-4 text-slate-800 rounded mx-1 text-sm capitalize"
              >
                {{ ability }}
              </span>
            </p>
            <p class="text-gray-400">
              <strong>XP:</strong> {{ pokemon.experience }} Exp
            </p>
          </div>
          <div v-if="activeTab === 'moves'" class="mt-4 space-y-4">
          <div
            class="grid grid-cols-2 max-h-[300px] overflow-y-auto p-2 md:grid-cols-3 gap-4"
          >
            <span
              v-for="move in pokemon.moves"
              :key="move.move.name"
              class="bg-secondary rounded-md text-center text-teks font-medium text-sm"
            >
              {{ move.move.name }}
            </span>
          </div>
        </div>
        <div v-if="activeTab === 'stats'" class="mt-4 space-y-2">
          <div v-for="stat in pokemon.stats" :key="stat.stat.name">
            <p class="text-teks font-semibold">
              {{ stat.stat.name.toUpperCase() }}: {{ stat.base_stat }}
            </p>
            <div class="w-full bg-white rounded-full h-3">
              <div
                class="bg-yellow-400 h-3 rounded-full transition-all duration-300"
                :style="{ width: stat.base_stat + '%' }"
              ></div>
            </div>
            </div>
            </div>
        </div>

        <div class="desktop-buttons">
          <button
            @click="goToNextPokemon"
            class="btn btn-neutral size-10 rounded-full m-auto ml-5 pl-4 pr-5"
          >
            >
          </button>
        </div>
      </div>

      <div class="mobile-buttons mt-4 flex justify-center space-x-4">
        <button
          @click="goToPreviousPokemon"
          class="btn btn-neutral size-10 rounded-full pl-4 pr-3"
        >
          <
        </button>
        <button
          @click="goToNextPokemon"
          class="btn btn-neutral size-10 rounded-full pl-4 pr-5"
        >
          >
        </button>
      </div>
    </div>
  </div>

  <div
    v-if="caughtMessage"
    class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 animate-fadeIn"
  >
    <div class="bg-white p-6 rounded-lg shadow-lg text-center w-96 relative">
      <button
        @click="closeMessage"
        class="absolute top-2 right-2 text-gray-500 hover:text-gray-700"
      >
        ✖
      </button>

      <div v-if="caughtMessage.success" class="animate-fadeIn">
        <h2 class="text-xl font-semibold text-green-700">
          horeee dapet {{ caughtMessage.pokemon.name }}!
        </h2>
        <img
          :src="caughtMessage.pokemon.image"
          alt="Pokemon"
          class="w-32 mx-auto my-2"
        />
        <p class="text-gray-600">
          Type: {{ caughtMessage.pokemon.types.join(", ") }}
        </p>
      </div>
      <div v-else class="animate-fadeIn">
        <h2 class="text-xl font-semibold text-red-700">
          Pokemon Mu kabur......
        </h2>
      </div>

      <button
        @click="closeMessage"
        class="mt-4 px-4 py-2 bg-red-500 rounded hover:bg-red-600 text-white"
      >
        Close
      </button>
    </div>
  </div>
</template>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
.animate-fadeIn {
  animation: fadeIn 0.3s ease-in-out;
}

.mobile-buttons {
  display: none;
}

.desktop-buttons {
  display: flex;
}

@media (max-width: 768px) {
  .mobile-buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-top: 1rem;
  }

  .desktop-buttons {
    display: none;
  }
}
</style>