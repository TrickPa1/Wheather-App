<template>
  <q-page :class="['flex column relative-position overflow-hidden', className = classeFundo]">
    
    <!-- Barra de Pesquisa -->
    <div class=" col fixed-top z-top q-mt-sm">
      <div class="search-wrapper shadow-custom q-mt-xl q-mx-auto">
        <q-input
          v-model="search"
          @keyup.enter="getWeatherBySearch"
          placeholder="Search"
          borderless
          dense
          dark
          class="search-input col"
        >
          <template v-slot:before>
            <q-icon 
              @click="getLocation"
              name="fmd_good" 
              class="location-icon cursor-pointer"
            />
          </template>

          <template v-slot:append>
            <q-btn
              @click="getWeatherBySearch" 
              round 
              dense 
              unelevated
              icon="search"
              class="search-button"
            />
          </template>
        </q-input>
     </div>
   </div>

   <!-- Temperatura -->
   <template v-if="weatherData"> 
      <div class="col text-white text-center corpo">

        <!-- Animação do lottie-->
        <div style="max-width center: 300px; width: 100%;">
          <Vue3Lottie 
            :animationData="lottieSelecionado" 
            :height="300" 
            :width="300" 
            :loop="true"
            :autoPlay="true"
          />
        </div>

        <!-- Nome da Cidade -->
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <!-- Estado da Temperatura -->
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main}}
        </div>

        <!-- A temperatura-->
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>

        <!-- os detalhes extra, malta velocidade, humidade e nuvens(muda de acordo as condições climaticas(chuva, neve, normal(somente a nuvem)))-->
        <div class="weather-details-card row items-center justify-around q-py-md q-px-sm q-mt-lg">
          
          <!-- O vento-->
          <div class="col text-center detail-column">
            <q-icon name="air" size="28px" class="text-amber-5 text-shadow-subtle q-mb-xs" />
            <div class="text-weight-bold text-body1">
              {{ Math.round(weatherData.wind.speed * 3.6) }} <span class="text-caption text-weight-light">km/h</span>
            </div>
            <div class="text-caption text-grey-4 opacity-70">Vento</div>
          </div>

          <div class="vertical-divider"></div>
          
          <!-- humidade -->
          <div class="col text-center detail-column">
            <q-icon name="water_drop" size="28px" class="text-blue-4 text-shadow-subtle q-mb-xs" />
            <div class="text-weight-bold text-body1">
              {{ weatherData.main.humidity }}<span class="text-caption text-weight-light">%</span>
            </div>
            <div class="text-caption text-grey-4 opacity-70">Humidade</div>
          </div>

          <div class="vertical-divider"></div>
          
          <!-- Nuvens -->
          <div class="col text-center detail-column">
            
            <!-- Para caso de chuva-->
            <template v-if="weatherData.rain">
              <q-icon name="umbrella" size="28px" class="text-blue-3 text-shadow-subtle q-mb-xs" />
              <div class="text-weight-bold text-body1">
                {{ weatherData.rain['1h'] || weatherData.rain['3h'] || 0 }}<span class="text-caption text-weight-light"> mm</span>
              </div>
              <div class="text-caption text-grey-4 opacity-70">Chuva</div>
            </template>

            <!-- Para caso de neve-->
            <template v-else-if="weatherData.snow">
              <q-icon name="ac_unit" size="28px" class="text-light-blue-2 text-shadow-subtle q-mb-xs" />
              <div class="text-weight-bold text-body1">
                {{ weatherData.snow['1h'] || weatherData.snow['3h'] || 0 }}<span class="text-caption text-weight-light"> mm</span>
              </div>
              <div class="text-caption text-grey-4 opacity-70">Neve</div>
            </template>
            
            <!-- Para casos normais(sem chuva ou neve) -->
            <template v-else>
              <q-icon name="cloud" size="28px" class="text-grey-4 text-shadow-subtle q-mb-xs" />
              <div class="text-weight-bold text-body1">
                {{ weatherData.clouds ? weatherData.clouds.all : 0 }}<span class="text-caption text-weight-light">%</span>
              </div>
              <div class="text-caption text-grey-4 opacity-70">Nuvens</div>
            </template>
          </div>
        </div>

        <!-- Revisoes em Horas -->
        <div class="hourly-forecast-card q-pa-md q-mt-md">
          <div class="row no-wrap overflow-auto custom-scroll q-pb-sm">
            
            <div 
              v-for="(hora, index) in forecastData" 
              :key="index" 
              class="hourly-item flex column items-center q-px-sm"
            >
              <span class="text-caption text-weight-medium text-grey-3 text-nowrap text-center">
                {{ index === 0 ? 'Agora' : formatarHora(hora.dt_txt) }}
              </span>
              
              <q-img 
                :src="getIconeLocal(hora.weather[0].icon)" 
                style="width: 35px; height: 35px;" 
                class="q-my-xs"
                fit="contain"
              />
              
              <span class="text-body1 text-weight-bold text-white">
                {{ Math.round(hora.main.temp) }}&deg;
              </span>
            </div>
          </div>
      </div>
   </div>
   </template>

   <!-- Tela Inicial -->
   <template v-else>
    <div class="col column text-center text-white corpo2">
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
  <div class="borda-desfocada"></div>
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
const forecastData = ref([])

const lat = ref(null)
const lon = ref(null)

const apiUrl = 'https://api.openweathermap.org/data/2.5/weather'
const apiUrlForecast = 'https://api.openweathermap.org/data/2.5/forecast'
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

//icones 
const getIconeLocal = (codigoIcone) => {
  return new URL(`../assets/icones/${codigoIcone}.png`, import.meta.url).href
}

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
  fetch(`${apiUrl}?lat=${lat.value}&lon=${lon.value}&appid=${apiKey}&units=metric`)
    .then(response => {
      if (!response.ok) throw new Error('Erro na requisição')
      return response.json() // Converte a resposta para JSON
    })
    .then(data => {
      weatherData.value = data
      getHourlyForecast()
    })
    .catch(error => {
      console.error('Erro ao buscar clima:', error)
    })
}

const getWeatherBySearch = () => {
  fetch(`${apiUrl}?q=${search.value}&appid=${apiKey}&units=metric`)
    .then(response => {
      if (!response.ok) throw new Error('Erro na requisição')
      return response.json() // Converte a resposta para JSON
    })
    .then(data => {
      weatherData.value = data
      getHourlyForecast(search.value)
    })
    .catch(error => {
      console.error('Erro ao buscar clima:', error)
    })
}

const getHourlyForecast = (cidade) => {
  // Se for por coordenadas, altere a URL para lat/lon igual fez no getWeatherByCoords
  const url = cidade 
    ? `${apiUrlForecast}?q=${cidade}&appid=${apiKey}&units=metric`
    : `${apiUrlForecast}?lat=${lat.value}&lon=${lon.value}&appid=${apiKey}&units=metric`

  fetch(url)
    .then(res => res.json())
    .then(data => {
      // A API gratuita retorna de 3 en 3 horas. Pegamos os 8 primeiros resultados (24 horas)
      forecastData.value = data.list.slice(0, 8)
    })
    .catch(err => console.error("Erro ao buscar previsão horária:", err))
}

const formatarHora = (dataString) => {
  if (!dataString) return ''
  const data = new Date(dataString)
  let horas = data.getHours()
  const ampm = horas >= 12 ? 'PM' : 'AM'
  horas = horas % 12
  horas = horas ? horas : 12 // O número 0 deve ser virar 12
  return `${horas} ${ampm}`
}

</script>

