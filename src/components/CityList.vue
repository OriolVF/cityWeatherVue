<template>
    <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)"></CityCard>
    </div>
    <p v-if="savedCities.length === 0">
        No locations added. To track a location search and add in the field above.
    </p>
</template>

<script setup>
import CityCard from './CityCard.vue';
import { ref } from "vue";
import axios from 'axios';
import { useRouter } from 'vue-router';

const savedCities = ref([]);
const getCities = async () => {
    if (localStorage.getItem('savedCities')) {
        savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
        console.log(savedCities.value.length);
        const requests = [];
        savedCities.value.forEach((city) => {
            requests.push(
                axios.get(
                    `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid={appid}&units=metric`)
            );
        });

        const weatherData = await Promise.all(requests);
        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data;
        });
    }
}
await getCities();

const router = useRouter();
const goToCityView = (city) => {
    router.push({
        name: 'cityView',
        params:{
            state: city.state, 
            city: city.city
        },
        query:{
            lat: city.coords.lat,
            lng: city.coords.lng
        }
    })
};

</script>