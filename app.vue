<script setup lang="ts">
const { offset } = useRoute().query;
const count = ref(Number(offset) || 0);
const { status, data, error } = await useAsyncData(
  `pokemons`,
  async () => {
    try {
      const pokemons = await $fetch(
        `https://pokeapi.co/api/v2/pokemon?offset=${count.value}&limit=20`
      );
      const pokemonsWithDetails = await Promise.allSettled(
        pokemons.results.map(
          async ({ url }: { url: string }) => await $fetch(url)
        )
      );
      return pokemonsWithDetails;
    } catch (e) {
      console.log(e);
      throw e;
    }
  },
  { watch: [count] }
);
</script>

<template>
  <NuxtLink :href="'?offset=' + (count + 20)" @click="count += 20"
    >Next</NuxtLink>
  <div v-if="status === 'pending'">Loading ...</div>
  <div v-else>
    <div v-for="pokemon in data">
      {{ pokemon.value.name }}
    </div>
  </div>
</template>
