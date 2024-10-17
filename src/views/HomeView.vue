<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or province"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="searchResults"
      >
        <p v-if="searchError">Somthing went wrong, please try again.</p>
        <p v-if="!searchError && searchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in searchResults"
            :key="searchResult.lat"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.name }}, {{ searchResult.state }},
            {{ searchResult.country }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <p>Loading...</p>
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';
import CityList from '@/components/CityList.vue';

const router = useRouter();
const searchQuery = ref('');
const queryTimeout = ref(null);
const searchResults = ref(null);
const searchError = ref(null);

const previewCity = searchResult => {
  const city = searchResult.name;
  const state = searchResult.state;
  router.push({
    name: 'cityView',
    params: { state: state, city: city },
    query: {
      lat: searchResult.lat,
      lon: searchResult.lon,
      preview: true,
    },
  });
};

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
        const result = await axios.get(
          `http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=5&appid=${import.meta.env.VITE_OPEN_MAPS_API_KEY}`,
        );
        searchResults.value = result.data;
      } catch {
        searchError.value = true;
      }
      return;
    }
    searchResults.value = null;
  }, 300);
};
</script>
