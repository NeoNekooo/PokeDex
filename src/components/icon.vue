<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { RouterLink } from "vue-router";
const caughtCount = ref(0);
</script>

<template>
  <div class="navbar absolute py-4 bg-transparent px-6 w-full">
    <div class="flex items-center justify-between w-full">
      <router-link to="/">
        <img src="../assets/poke.png" class="w-[105px]" alt="pokemon" />
      </router-link>
    </div>
  </div>
</template>

<style>
.poke {
  width: 30px;
}
</style>
