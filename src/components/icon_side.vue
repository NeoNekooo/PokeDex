<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { RouterLink } from "vue-router";
const capturePokemon = ref(0);

const updateCount = () => {
  const storedPokemon = JSON.parse(localStorage.getItem("pokemonList")) || [];
  capturePokemon.value = storedPokemon.length;
};

onMounted(() => {
  updateCount();
  window.addEventListener("storage", updateCount);
});

onUnmounted(() => {
  window.removeEventListener("storage", updateCount);
});
</script>

<template>
  <div class="h-screen flex bg-transparent absolute">
    <nav
      class="fixed left-0 top-0 h-full bg-transparent py-5 px-3 w-16 flex flex-col justify-center items-center space-y-6 pointer-events-none"
    >
      <RouterLink to="/capture" class="w-16 rounded ml-4">
        <div
          class="lg:tooltip-right lg:tooltip lg:tooltip-info"
          :data-tip="capturePokemon + ' ' + 'catch'"
        >
          <a
            href="#"
            class="text-gray-500 hover:text-blue-500 transition pointer-events-auto"
          >
            <img src="../assets/pokeballs.png" alt="" class="animate-spin" />
          </a>
        </div>
      </RouterLink>
    </nav>
  </div>
</template>

<style scoped></style>
