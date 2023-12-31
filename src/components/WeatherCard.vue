<template>
    <!-- Weather Card Component Markup -->
    <div class="bg-orange-100 p-7 text-orange-400 rounded-3xl col-span-3 md:row-span-2 w-full">
        <div v-if="typeof weather.main !== 'undefined'">

            <!-- Today and Arrow button -->
            <div class="flex items-center justify-center mb-4 text-orange-400">
                <div class="text-xl font-bold mr-2">Today</div>
                <svg width="50px" height="50px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M7 10L12 15L17 10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"
                        stroke-linejoin="round" />
                </svg><!-- Arrow button, adjust size as needed -->
            </div>


            <!-- Temperature and Weather Symbol -->
            <div class="text-orange-300 text-5xl flex items-center justify-center">
                <div class="text-5xl ml-2">
                    {{ getWeatherSymbol(weather.weather[0].icon) }}
                </div>
                <div class="text-orange-400 text-8xl font-medium p-5">
                    {{ Math.round(weather.main.temp) }}°
                </div>

            </div>

            <!-- Weather Status -->
            <div class="text-xl font-medium p-4 text-center">
                {{ weather.weather[0].main }}
            </div>

            <!-- Location and Date -->
            <div class="mb-2 text-center">
                <div class="text-lg p-4">{{ weather.name }}, {{ weather.sys.country }}</div>
                <div class="text-lg p-4">{{ dateBuilder() }}</div>
            </div>

            <!-- Sunset Time and Avg Temp -->
            <div class="text-lg  text-center">
                Sunset: {{ formatTime(weather.sys.sunset) }} | Avg Temp: {{ Math.round(weather.main.temp) }}°c
            </div>

        </div>
    </div>
</template>

<script>
export default {
    props: {
        weather: Object,
        formatTime: Function,
        getWeatherSymbol: Function,
        dateBuilder: Function,
    },
}
</script>

<style>
/* Styles for Weather Card Component */
</style>
