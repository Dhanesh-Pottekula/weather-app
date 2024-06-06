<template>
  <div class="flex flex-col flex-1 items-center min-h-screen bg-cover bg-center bg-fixed" style="background-image: url('https://images.unsplash.com/photo-1506748686214-e9df14d4d9d0?ixlib=rb-1.2.1&auto=format&fit=crop&w=1950&q=80');">
    <!-- Banner -->
    <div v-if="route.query.preview" class=" text-orange-400 p-4 bg-orange-100 w-full text-center">
      <p>Click on remove city Button in Bottom to go Back</p>
    </div>

    <!-- Weather Overview and Hourly Weather -->
    <div class="flex flex-col md:flex-row items-center md:items-start text-white py-12 w-full max-w-screen-lg gap-8">
      <!-- Weather Overview -->
      <div class="flex flex-col items-center md:items-start text-center md:text-left w-2/3 md:w-[30%] bg-orange-600 p-10 md:mx-20 rounded-xl mb-20 mx-4 my-4">
        <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
        <p class="text-sm mb-10 text-yellow-600">
          {{
            new Date(weatherData.currentTime).toLocaleDateString("en-us", {
              weekday: "short",
              day: "2-digit",
              month: "long",
            })
          }}
          {{
            new Date(weatherData.currentTime).toLocaleTimeString("en-us", {
              timeStyle: "short",
            })
          }}
        </p>
        <p class="text-8xl mb-8">{{ Math.round(weatherData.current.temp) }}&deg;</p>
        <p>Feels like {{ Math.round(weatherData.current.feels_like) }} &deg;</p>
        <p class="capitalize text-orange-400">{{ weatherData.current.weather[0].description }}</p>
      </div>

      <!-- Hourly Weather -->
      <div class="flex flex-col md:flex-1 w-full md:w-[70%]">
        <div class="mx-8 text-white">
          <h2 class="mb-4">Hourly Weather</h2>
          <div class="flex gap-4 overflow-x-scroll">
            <div v-for="hourData in weatherData.hourly" :key="hourData.dt" class="flex flex-col gap-4 items-center">
              <p class="whitespace-nowrap text-md">
                {{
                  new Date(hourData.currentTime).toLocaleTimeString("en-us", {
                    hour: "numeric",
                  })
                }}
              </p>
              <img class="w-auto h-[50px] object-cover" :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`" alt="Hourly Weather Icon" />
              <p class="text-xl">{{ Math.round(hourData.temp) }}&deg;</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Remove City Button -->
    <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500" @click="removeCity">
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    // Calculate current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // Calculate hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();

const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter(
    (city) => city.id !== route.query.id
  );
  localStorage.setItem(
    "savedCities",
    JSON.stringify(updatedCities)
  );
  router.push({
    name: "home",
  });
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

body {
  font-family: 'Roboto', sans-serif;
  background: #f0f4f8;
}

.bg-weather-primary {
  background-color: #2c3e50; /* Replace with your desired color */
}

.bg-weather-secondary {
  background-color: #34495e; /* Replace with your desired color */
}

.text-weather-primary {
  color: #2c3e50;
}

.text-weather-secondary {
  color: #34495e;
}

.page-enter-active {
  transition: 600ms ease all;
}

.page-enter-from {
  opacity: 0;
}

.container {
  max-width: 600px;
}

@media (min-width: 768px) {
  .container {
    max-width: 800px;
  }
}

@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
  }
}
</style>
