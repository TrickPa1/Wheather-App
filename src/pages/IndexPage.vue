<template>
  <q-page :class="['flex column relative-position overflow-hidden', className = classeFundo]">
   <div class="col q-pt-lg q-px-md">
    <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon 
            @click="getLocation"
            name="fmd_good" 
          />
        </template>

        <template v-slot:append>
          <q-btn
            @click="getWeatherBySearch" 
            round 
            dense 
            flat 
            icon="search"
          />
        </template>
      </q-input>
   </div>

   <template v-if="weatherData"> 
      <div class="col text-white text-center">
        <div style="max-width center: 300px; width: 100%;">
          <Vue3Lottie 
            :animationData="lottieSelecionado" 
            :height="300" 
            :width="300" 
            :loop="true"
            :autoPlay="true"
          />
        </div>
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main}}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
   </template>
   <template v-else>
    <div class="col column text-center text-white">
      <div class="col text-h2 text-weight-thin">
        Weather<br>App
      </div>
      <q-btn
       @click="getLocation" 
       class="col" 
       flat
      >
      <q-icon left size="3em" name="fmd_good" />
      <div>Encontrar minha localização</div>
    </q-btn>
    </div>
   </template>  
  </q-page>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Vue3Lottie } from 'vue3-lottie'
import SunnyJson from 'src/assets/sunny.json'
import CloudyJson from 'src/assets/cloudy.json'
import RainJson from 'src/assets/rain.json'
import ThunderJson from 'src/assets/thunder.json'
import WindyJson from 'src/assets/windy.json'
import NightJson from 'src/assets/noite.json'
import CloudyNightJson from 'src/assets/noite_nublada.json'
import RainyNightJson from 'src/assets/noite_chuvosa.json'

// --- ESTADO REATIVO ---
const search = ref('')
const weatherData = ref(null)

const lat = ref(null)
const lon = ref(null)

const apiUrl = 'https://api.openweathermap.org/data/2.5/weather'
const apiKey = '8d87ded8c44d81d144c6b698ff62ea48'

//animações do clima
const lottieSelecionado = computed(() => {
  // Se ainda não carregou nada, mostra o tempo limpo/ensolarado
  if (!weatherData.value) return SunnyJson

  const condicaoClima = weatherData.value.weather[0].main
  const iconeClima = weatherData.value.weather[0].icon
  const isNight = iconeClima ? iconeClima.endsWith('n') : false

  if (isNight && condicaoClima === 'Clear') {
    return NightJson
  }

  if(isNight && condicaoClima == 'Clouds') {
    return CloudyNightJson
  }

  if(isNight && condicaoClima == 'Rain') {
    return RainyNightJson
  }

  // Seleção das animoções do JSON
  switch (condicaoClima) {
    case 'Clear':
      return SunnyJson
      
    case 'Clouds':
      return CloudyJson
      
    case 'Rain':
    case 'Drizzle':
      return RainJson
      
    case 'Thunderstorm':
      return ThunderJson
      
    // Para ventania, nevoeiro ou tempestades de areia/vento, tornado
    case 'Windy': 
    case 'Squall':
    case 'Tornado':
      return WindyJson
      
    default:
      //Para névoa leve (Mist/Fog)
      return CloudyJson 
  }
})

//animações do fundo
const classeFundo = computed(() => {
  if (!weatherData.value) return 'bg-sunny'

  const condicaoClima = weatherData.value.weather[0].main
  const iconeClima = weatherData.value.weather[0].icon
  const isNight = iconeClima ? iconeClima.endsWith('n') : false

  if (isNight && condicaoClima === 'Clear') return 'bg-night'

  switch (condicaoClima) {
    case 'Clear': return 'bg-sunny'
    case 'Clouds': return 'bg-cloudy'
    case 'Rain': 
    case 'Drizzle': return 'bg-cloudy'
    case 'Thunderstorm': return 'bg-thunder'
    default: return isNight ? 'bg-night' : 'bg-sunny'
  }
})

// --- MÉTODOS ---
const getLocation = () => {
  navigator.geolocation.getCurrentPosition(
    position => {
      console.log('position: ', position)
      lat.value = position.coords.latitude
      lon.value = position.coords.longitude
      getWeatherByCoords()
    },
    error => {
      console.error('Erro de geolocalização:', error)
    }
  )
}

const getWeatherByCoords = () => {
  // Usando o fetch nativo do navegador
  fetch(`${apiUrl}?lat=${lat.value}&lon=${lon.value}&appid=${apiKey}&units=metric`)
    .then(response => {
      if (!response.ok) throw new Error('Erro na requisição')
      return response.json() // Converte a resposta para JSON
    })
    .then(data => {
      weatherData.value = data
    })
    .catch(error => {
      console.error('Erro ao buscar clima:', error)
    })
}

const getWeatherBySearch = () => {
  // Usando o fetch nativo do navegador
  fetch(`${apiUrl}?q=${search.value}&appid=${apiKey}&units=metric`)
    .then(response => {
      if (!response.ok) throw new Error('Erro na requisição')
      return response.json() // Converte a resposta para JSON
    })
    .then(data => {
      weatherData.value = data
    })
    .catch(error => {
      console.error('Erro ao buscar clima:', error)
    })
}
</script>

