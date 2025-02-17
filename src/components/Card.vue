<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";

const pokemonList = ref([]);
const showCount = ref(20);
const router = useRouter();

const fetchPokemon = async () => {
  try {
    const promises = [];
    for (let i = 1; i <= 200; i++) {
      promises.push(
        fetch(`https://pokeapi.co/api/v2/pokemon/${i}`).then((res) =>
          res.json()
        )
      );
    }

    const results = await Promise.all(promises);
    pokemonList.value = results.map((pokemon) => ({
      id: pokemon.id,
      name: pokemon.name,
      image: pokemon.sprites.other["official-artwork"].front_default,
    }));
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

onMounted(fetchPokemon);

const showMore = () => {
  showCount.value += 20;
};

const goToDetail = (id) => {
  router.push(`/pokemon/${id}`);
};
</script>

<template>
  <section
    class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 p-4 mt-28"
  >
    <div v-if="isLoading" class="text-center text-lg font-semibold text-white">
      Loading...
    </div>
    <div
      v-for="pokemon in pokemonList.slice(0, showCount)"
      :key="pokemon.id"
      class="bg-gray-800 transition ease-in-out hover:-translate-y-2 shadow-lg rounded-xl p-4 flex flex-col items-center group"
      @click="goToDetail(pokemon.id)"
    >
      <div class="text-gray-500 text-sm font-semibold hover:text-blue-400 transition-colors">#{{ pokemon.id }}</div>
      <img
        :src="pokemon.image"
        alt="Pokemon"
        class="w-64 h-64 object-contain"
      />
      <h1 class="text-xl font-bold text-slate-300 mt-2 capitalize transition-colors group-hover:text-yellow-400">
        {{ pokemon.name }}
      </h1>
    </div>
  </section>

  <footer class="flex flex-col items-center justify-center py-4">
    <button
      @click="showMore"
      class="bg-gray-400 rounded-lg py-2 px-4 text-xl font-semibold text-slate-900 hover:bg-gray-500 transition"
    >
      Show more..
    </button>
  </footer>
</template>
