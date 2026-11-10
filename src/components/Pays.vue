<script setup lang="ts">
    import {ref, onMounted } from 'vue'
    const selectedP=ref('')
    const selectedV=ref('')
    const pays = ref([])
    const villes = ref([])

    const weather = ref(null)


    onMounted(async () => {
        const response = await fetch(
            'http://localhost/vuePay/pays.php'
        )

        pays.value = await response.json()
    })

    async function getV() {
        selectedV.value = ''
        villes.value = []

        const response = await fetch(
            `http://localhost/vuePay/villes.php?pays_id=${selectedP.value}`
        )

        villes.value = await response.json()
    }


async function getM() {

    try {
        const geoResponse = await fetch(
            `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(selectedV.value)}&count=1&language=en&format=json`
        )

        const geoData = await geoResponse.json()

        const lat = geoData.results[0].latitude
        const long = geoData.results[0].longitude
        const weatherResponse = await fetch(
            `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${long}&current=temperature_2m,relative_humidity_2m,apparent_temperature,wind_speed_10m,weather_code&timezone=auto`
        )
        const data = await weatherResponse.json()
        weather.value = data.current

    } catch(err) {
        weather.value = null

    }
}

    

</script>

<template #heading>
    <div>
        <label for="">Selected pays: </label><br>
        <select v-model="selectedP" @change="getV()" >
            <option v-for="p in pays" :key="p.id" :value="p.id">
                {{ p.nom }}
            </option>
        </select><br>
        <label for="">Selected villes: </label><br>
        <select v-model="selectedV" @change="getM()">
            <option v-for="v in villes" :key="v.nom" :value="v.nom" @change="getM()">
                {{ v.nom }}
            </option>
        </select>

        <div v-if="weather">

            <h2>Meteo</h2>

            <p>Temperature: {{ weather.temperature_2m }} °C</p>

            <p>Feels like: {{ weather.apparent_temperature }} °C</p>

            <p>Humidity: {{ weather.relative_humidity_2m }}%</p>

            <p>Wind speed: {{ weather.wind_speed_10m }} km/h</p>

        </div>

    </div>
</template>