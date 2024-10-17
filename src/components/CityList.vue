<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>
  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import CityCard from '@/components/CityCard.vue';
import { useRouter } from 'vue-router';

const savedCities = ref([]);
const getCities = async () => {
  const storedCities = localStorage.getItem('savedCities');
  if (storedCities) {
    savedCities.value = JSON.parse(storedCities);
    const requests = [];
    savedCities.value.forEach(city => {
      requests.push(
        axios.get(
          `http://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lon}&appid=${import.meta.env.VITE_OPEN_MAPS_API_KEY}&units=metric`,
        ),
      );
    });

    const weatherData = await Promise.all(requests);

    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value.data;
    });
  }
};
await getCities();

const router = useRouter();
const goToCityView = city => {
  router.push({
    name: 'cityView',
    params: {
      state: city.state,
      city: city.city,
    },
    query: { id: city.id, lat: city.coords.lat, lon: city.coords.lon },
  });
};
</script>
