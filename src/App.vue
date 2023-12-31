<template>
  <div id="app" :class="{ 'warm': typeof weather.main !== 'undefined' && weather.main.temp > 16 }">
    <main class="bg-[url('../public/images/scarecrow.png')] bg-no-repeat bg-cover">


      <div class="flex items-center justify-center p-5">
        <div class="rounded-lg bg-gray-200 p-5">
          <div class="flex">
            <div
              class="flex w-10 items-center justify-center rounded-tl-lg rounded-bl-lg border-r border-gray-200 bg-white p-5">
              <svg viewBox="0 0 20 20" aria-hidden="true"
                class="pointer-events-none absolute w-5 fill-gray-500 transition">
                <path
                  d="M16.72 17.78a.75.75 0 1 0 1.06-1.06l-1.06 1.06ZM9 14.5A5.5 5.5 0 0 1 3.5 9H2a7 7 0 0 0 7 7v-1.5ZM3.5 9A5.5 5.5 0 0 1 9 3.5V2a7 7 0 0 0-7 7h1.5ZM9 3.5A5.5 5.5 0 0 1 14.5 9H16a7 7 0 0 0-7-7v1.5Zm3.89 10.45 3.83 3.83 1.06-1.06-3.83-3.83-1.06 1.06ZM14.5 9a5.48 5.48 0 0 1-1.61 3.89l1.06 1.06A6.98 6.98 0 0 0 16 9h-1.5Zm-1.61 3.89A5.48 5.48 0 0 1 9 14.5V16a6.98 6.98 0 0 0 4.95-2.05l-1.06-1.06Z">
                </path>
              </svg>
            </div>
            <input type="text" class="w-full max-w-[860px] bg-white pl-2 text-base font-semibold outline-0" placeholder=""
              id="" v-model="query" @keypress="fetchWeather" />
          </div>
        </div>
      </div>


      <div class="grid grid-cols-1 md:grid-cols-8 md:grid-rows-2 gap-14">



        <WeatherCard v-if="typeof weather.main !== 'undefined'" :weather="weather" :formatTime="formatTime"
          :getWeatherSymbol="getWeatherSymbol" :dateBuilder="dateBuilder" />

        <WeatherListCard v-if="typeof forecast.list !== 'undefined'" :forecast="forecast"
          :formatForecastTime="formatForecastTime" :getWeatherSymbol="getWeatherSymbol" />

        <WeatherText />



      </div>

      <div v-if="error" class="text-red-700 text-center font-bold text-xl">{{ error }}</div>
    </main>
  </div>
</template>
<script>


import WeatherCard from './components/WeatherCard.vue';
import WeatherListCard from './components/WeatherListCard.vue';
import WeatherText from './components/WeatherText.vue'

import config from './config.js';


export default {
  name: 'App',
  components: {
    WeatherCard,
    WeatherListCard,
    WeatherText
  },
  data() {
    return {
      api_key: config.api_key,
      url_base: config.url_base,
      query: '',
      weather: {},
      forecast: {},
      error: null
    }
  },


  methods: {

    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      const hours = date.getHours();
      const minutes = date.getMinutes();
      return `${hours}:${minutes < 10 ? '0' : ''}${minutes}`;
    },

    getWeatherSymbol(weatherCode) {
      const weatherSymbols = {
        '01d': '☀️',
        '01n': '🌙',
        '02d': '⛅',
        '02n': '☁️',
        '03d': '☁️',
        '03n': '☁️',
        '04d': '☁️',
        '04n': '☁️',
        '09d': '🌧️',
        '09n': '🌧️',
        '10d': '🌦️',
        '10n': '🌦️',
        '11d': '⛈️',
        '11n': '⛈️',
        '50d': '🌫️',
        '50n': '🌫️',
      };

      return weatherSymbols[weatherCode] || '❓';
    },


    fetchWeather(e) {
      if (e.key === 'Enter') {
        this.error = null; // Clear previous errors
        fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
          .then(res => res.json())
          .then(this.setWeather)
          .catch(this.handleFetchError);
      }
    },

    setWeather(weatherData) {
      this.weather = weatherData;
      this.fetchForecast(weatherData.coord);
    },

    formatForecastTime(timestamp) {
      const date = new Date(timestamp * 1000);
      const hours = date.getHours();
      const minutes = date.getMinutes();
      const ampm = hours >= 12 ? 'PM' : 'AM';
      const formattedHours = hours % 12 || 12; // Convert to 12-hour format
      return `${formattedHours}:${minutes < 10 ? '0' : ''}${minutes} ${ampm}`;
    },

    fetchForecast(coord) {
      fetch(`${this.url_base}forecast?lat=${coord.lat}&lon=${coord.lon}&units=metric&APPID=${this.api_key}`)
        .then(res => res.json())
        .then(forecastData => {
          console.log('Forecast Data:', forecastData);

          // Get the next 10 data points from the forecast
          const next10Forecast = forecastData.list.slice(1, 11);

          this.forecast = { list: next10Forecast };
        })
        .catch(this.handleFetchError);
    },


    handleFetchError(error) {
      this.error = 'Error fetching data. Please try again.';
      console.error('Error:', error);
    },

    dateBuilder() {
      let d = new Date();
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

      return `${days[d.getDay()]} ${d.getDate()} ${months[d.getMonth()]} ${d.getFullYear()}`;
    },

    formatForecastDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return `${date.toLocaleDateString(undefined, { weekday: 'short' })}, ${date.toLocaleTimeString(undefined, { hour: 'numeric', minute: 'numeric' })}`;
    },

    async getInitialLocation() {
      try {
        const position = await this.getCurrentPosition();
        const { latitude, longitude } = position.coords;
        this.fetchWeatherByCoords(latitude, longitude);
      } catch (error) {
        console.error('Error getting current location:', error);
        this.error = 'Error getting current location. Please enter a location manually.';
      }
    },

    getCurrentPosition() {
      return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(resolve, reject);
      });
    },

    async fetchWeatherByCoords(latitude, longitude) {
      this.error = null; // Clear previous errors
      try {
        const response = await fetch(
          `${this.url_base}weather?lat=${latitude}&lon=${longitude}&units=metric&APPID=${this.api_key}`
        );
        const weatherData = await response.json();
        this.setWeather(weatherData);
        this.fetchForecast(weatherData.coord);
      } catch (error) {
        this.handleFetchError(error);
      }
    },
  },
  mounted() {
    this.getInitialLocation();
  },

};


</script>

<style>
@tailwind base;
@tailwind components;
@tailwind utilities;

@import url('https://fonts.googleapis.com/css2?family=DM+Serif+Display&family=Poppins:wght@300;400;500;700&display=swap');


main {
  font-family: 'Poppins', sans-serif;
}

main {
  min-height: 100vh;
  padding: 25px;

}
</style>
