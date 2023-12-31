<template>
    <!-- Weather List Card Component Markup -->
    <div class="bg-orange-200 bg-opacity-50 backdrop-blur text-white col-span-3 md:col-span-5 grid grid-cols-5 gap-4 rounded-3xl p-10 blur-bg"
        v-if="typeof forecast.list !== 'undefined'">
        <div v-for="(day) in forecast.list.slice(0, 10)" :key="day.dt" class="">
            <div class="flex flex-col items-center">
                <div class="md:text-lg">
                    {{ formatForecastTime(day.dt) }}
                </div>
                <div class="flex items-center">
                    <div class="">
                        {{ getWeatherSymbol(day.weather[0].icon) }}
                    </div>
                    <div class="">
                        {{ Math.round(day.main.temp) }}°c
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        forecast: Object,
        formatForecastTime: Function,
        getWeatherSymbol: Function,
    },
}
</script>

<style>
/* Styles for Weather List Card Component */
</style>
