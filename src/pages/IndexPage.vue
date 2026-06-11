<template>
  <q-page :class="['flex column relative-position overflow-hidden', classeFundo]">
    
    <q-carousel
      v-model="slideAtual"
      transition-prev="slide-right"
      transition-next="slide-left"
      :swipeable="!!weatherData"
      animated
      class="bg-transparent full-page-carousel col"
    >
      
      <!-- Carousel Para Mapa de Vento-->
      <q-carousel-slide name="vento" class="q-pa-md text-white flex flex-center">
        <div class="full-width">
          
          <div class="text-h5 text-weight-bold q-mt-xl q-mb-md flex items-center q-gutter-sm">
            <q-icon name="air" class="text-amber-5 animate-pulse" />
            <span>Mapa de Vento</span>
          </div>
          
          <div class="map-container shadow-custom">
            <iframe 
              :src="urlMapaVento"
              frameborder="0"
              class="windy-iframe"
              allow="geolocation"
            ></iframe>
          </div>

        </div>
      </q-carousel-slide>

      <!-- Carousel Principal -->
      <q-carousel-slide name="home" class="q-pa-none q-mt-xl container-scroll-vertical">
        <!-- Barra de Pesquisa --> 
        <div class="fixed-top z-top barra-pesquisa-fixa q-pa-md q-mt-xl">
          <div class="search-wrapper shadow-custom q-mx-auto">
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
          <div class="text-white text-center corpo q-pt-xl q-px-md">
            
            <!-- Animação do lottie-->
            <div class="flex justify-center q-mx-auto" style="max-width: 300px; width: 100%;">
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
              {{ weatherData.weather[0].main }}
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
            
            <!-- Previsoes em Horas -->
            <div class="hourly-forecast-card q-pa-md q-mt-md q-mb-xl">
              <div class="row no-wrap overflow-auto custom-scroll q-pb-sm">
                <div 
                  v-for="(hora, index) in previsaoHoraria" 
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
      </q-carousel-slide>

      <!-- Carousel de Previsao detalhada-->
      <q-carousel-slide name="previsao" class="q-pa-md text-white container-scroll-vertical">
        
        <div class="text-h5 text-weight-bold q-mt-xl q-mb-md flex items-center q-gutter-sm">
          <q-icon name="equalizer" class="text-blue-4" />
          <span>Previsão Estendida</span>
        </div>

        <div v-if="dadosAmanhaCard" class="tomorrow-summary-card q-pa-lg q-mb-lg shadow-custom">
          
          <div class="row items-center justify-between no-wrap">
            <div class="col-5 flex flex-center">
              <q-img 
                :src="getIconeLocal(dadosAmanhaCard.icon)" 
                style="width: 110px; height: 110px; filter: drop-shadow(0px 8px 12px rgba(0,0,0,0.15));" 
                fit="contain"
              />
            </div>

            <div class="col-7 column items-start q-pl-md">
              <span class="text-subtitle1 text-weight-bold text-grey-3 opacity-90">Amanhã</span>
              <div class="row items-baseline text-white q-my-xs">
                <span class="text-h2 text-weight-medium">{{ dadosAmanhaCard.max }}</span>
                <span class="text-h5 text-grey-4 text-weight-light q-ml-xs">/{{ dadosAmanhaCard.min }}&deg;</span>
              </div>
              <span class="text-body2 text-weight-medium text-grey-3 capitalize">
                {{ dadosAmanhaCard.description }}
              </span>
            </div>
          </div>

          <q-separator dark class="q-my-lg" style="opacity: 0.08;" />

          <div class="row items-center justify-around text-center">
            <div class="col column items-center">
              <q-icon name="air" size="24px" class="text-grey-4 q-mb-xs" />
              <span class="text-body1 text-weight-bold">{{ dadosAmanhaCard.wind }}km/h</span>
              <span class="text-caption text-grey-5">Vento</span>
            </div>

            <div class="col column items-center">
              <q-icon name="cloud_queue" size="24px" class="text-grey-4 q-mb-xs" />
              <span class="text-body1 text-weight-bold">{{ dadosAmanhaCard.chuvaChance }}%</span>
              <span class="text-caption text-grey-5">Chuva</span>
            </div>

            <div class="col column items-center">
              <q-icon name="water_drop" size="24px" class="text-grey-4 q-mb-xs" />
              <span class="text-body1 text-weight-bold">{{ dadosAmanhaCard.humidity }}%</span>
              <span class="text-caption text-grey-5">Humidade</span>
            </div>
          </div>

        </div>

        <div class="forecast-days-card q-pa-md q-mb-xl" v-if="previsaoDiaria.length > 0">
          <div 
            v-for="(dia, index) in previsaoDiaria" 
            :key="index" 
            class="row items-center justify-between q-py-md daily-row"
          >
            <span class="text-body1 text-weight-bold text-capitalize col-3">
              {{ index === 0 ? 'Amanhã' : dia.nomeDia }}
            </span>
            <div class="col-2 text-center flex flex-center">
              <q-img :src="getIconeLocal(dia.icon)" style="width: 30px; height: 30px;" fit="contain" />
            </div>
            <div class="row items-center justify-end col-7">
              <span class="text-grey-4 text-right" style="width: 35px;">{{ Math.round(dia.min) }}&deg;</span>
              <div class="temp-bar-ios"></div>
              <span class="text-weight-bold text-right" style="width: 35px;">{{ Math.round(dia.max) }}&deg;</span>
            </div>
          </div>
        </div>
      </q-carousel-slide>
    </q-carousel>
    
    <!-- Pontinhos indicadores do Carousel-->
    <div v-if="weatherData" class="fixed-bottom row justify-center q-pb-lg pointer-events-none" style="z-index: 10;">
      <div class="ios-capsule-indicator flex items-center justify-center"> 
         <q-icon 
          name="near_me" 
          class="indicator-item arrow" 
          :class="{ 'active': slideAtual === 'vento' }" 
        />
        <div class="indicator-item dot" :class="{ 'active': slideAtual === 'home' }"></div>
        <div class="indicator-item dot" :class="{ 'active': slideAtual === 'previsao' }"></div>
        
      </div>
    </div>
    
    <!-- Efeito para borda desfocada -->
    <div class="borda-desfocada"></div>

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

//carousel
const slideAtual = ref('home')

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
  const url = cidade 
    ? `${apiUrlForecast}?q=${cidade}&appid=${apiKey}&units=metric`
    : `${apiUrlForecast}?lat=${lat.value}&lon=${lon.value}&appid=${apiKey}&units=metric`

  fetch(url)
    .then(res => res.json())
    .then(data => {
      forecastData.value = data.list
    })
    .catch(err => console.error("Erro ao buscar previsão horária:", err))
}

const previsaoHoraria = computed(() => {
  const lista = Array.isArray(forecastData.value) 
    ? forecastData.value 
    : (forecastData.value?.list || [])
    
  return lista.slice(0, 8) // Pega apenas as próximas 24h (8 blocos de 3h)
})

const formatarHora = (dataString) => {
  if (!dataString) return ''
  const data = new Date(dataString)
  let horas = data.getHours()
  const ampm = horas >= 12 ? 'PM' : 'AM'
  horas = horas % 12
  horas = horas ? horas : 12 // O número 0 deve ser virar 12
  return `${horas} ${ampm}`
}

const urlMapaVento = computed(() => {
  let lat
  let lon 
  if (weatherData.value && weatherData.value.coord) {
    lat = weatherData.value.coord.lat
    lon = weatherData.value.coord.lon
  }

  // URL do Windy configurado para focar no Vento (overlay=wind) e sem controlos pesados
  return `https://embed.windy.com/embed2.html?lat=${lat}&lon=${lon}&zoom=7&level=surface&overlay=wind&menu=&message=&marker=&type=map&location=coordinates`
})

//dados para previsões futuras
const dadosAmanhaCard = computed(() => {
  // Extrai a lista de forma ultra segura, seja ela um array direto ou dentro de .list
  const lista = Array.isArray(forecastData.value) 
    ? forecastData.value 
    : (forecastData.value?.list || [])

  // Se a lista ainda estiver vazia (antes da API responder), sai graciosamente sem quebrar
  if (lista.length === 0) return null

  // Usamos a data do primeiríssimo bloco para saber qual é o dia de "Hoje" na API
  if (!lista[0]?.dt_txt) return null
  const hojeString = lista[0].dt_txt.split(' ')[0]
  
  // Filtra apenas os blocos dos dias seguintes (remove o dia de hoje)
  const blocosFuturos = lista.filter(bloco => bloco.dt_txt && !bloco.dt_txt.includes(hojeString))
  if (blocosFuturos.length === 0) return null

  // Identifica o dia de amanhã exato
  const dataAmanhaString = blocosFuturos[0].dt_txt.split(' ')[0]
  const blocosAmanha = blocosFuturos.filter(bloco => bloco.dt_txt && bloco.dt_txt.startsWith(dataAmanhaString))
  if (blocosAmanha.length === 0) return null

  let maxTemp = -99
  let minTemp = 99
  let somaVento = 0
  let somaHumidade = 0
  let somaNuvens = 0
  
  // Escolhe por padrão o bloco do meio do dia
  let blocoPrincipal = blocosAmanha[Math.floor(blocosAmanha.length / 2)] || blocosAmanha[0]

  blocosAmanha.forEach(bloco => {
    if (bloco.main?.temp_max > maxTemp) maxTemp = bloco.main.temp_max
    if (bloco.main?.temp_min < minTemp) minTemp = bloco.main.temp_min
    somaVento += bloco.wind?.speed || 0
    somaHumidade += bloco.main?.humidity || 0
    somaNuvens += bloco.clouds?.all || 0
    
    // Tenta capturar a condição do meio da tarde (12h ou 15h) para o ícone ser realista
    if (bloco.dt_txt?.includes('12:00') || bloco.dt_txt?.includes('15:00')) {
      blocoPrincipal = bloco
    }
  })

  const totalBlocos = blocosAmanha.length || 1

  return {
    max: maxTemp === -99 ? 0 : Math.round(maxTemp),
    min: minTemp === 99 ? 0 : Math.round(minTemp),
    icon: blocoPrincipal.weather?.[0]?.icon || '01d',
    description: blocoPrincipal.weather?.[0]?.main || 'Clear',
    wind: Math.round((somaVento / totalBlocos) * 3.6), // Converte m/s para km/h
    humidity: Math.round(somaHumidade / totalBlocos),
    chuvaChance: blocoPrincipal.pop !== undefined ? Math.round(blocoPrincipal.pop * 100) : Math.round(somaNuvens / totalBlocos)
  }
})

// 2. Lista da previsao dos proximos dias
const previsaoDiaria = computed(() => {
  const lista = Array.isArray(forecastData.value) 
    ? forecastData.value 
    : (forecastData.value?.list || [])

  if (lista.length === 0) return []

  const diasAgrupados = {}

  lista.forEach(bloco => {
    if (!bloco.dt_txt) return
    const dataChave = bloco.dt_txt.split(' ')[0]
    
    if (!diasAgrupados[dataChave]) {
      diasAgrupados[dataChave] = {
        dataRaw: dataChave,
        min: bloco.main?.temp_min ?? 0,
        max: bloco.main?.temp_max ?? 0,
        icon: bloco.weather?.[0]?.icon || '01d'
      }
    } else {
      if (bloco.main?.temp_min < diasAgrupados[dataChave].min) {
        diasAgrupados[dataChave].min = bloco.main.temp_min
      }
      if (bloco.main?.temp_max > diasAgrupados[dataChave].max) {
        diasAgrupados[dataChave].max = bloco.main.temp_max
      }
    }
  })

  return Object.values(diasAgrupados).map(dia => {
    // Substituição de hífen por barra para garantir compatibilidade de datas no iOS/Safari
    const dataFormatada = dia.dataRaw.replace(/-/g, '/')
    const dataObjeto = new Date(dataFormatada)
    
    let nomeDia = '---'
    if (!isNaN(dataObjeto.getTime())) {
      nomeDia = dataObjeto.toLocaleDateString('pt-PT', { weekday: 'short' })
      nomeDia = nomeDia.replace('.', '')
    }

    return {
      ...dia,
      nomeDia: nomeDia
    }
  }).slice(1, 6) // Pega os 5 dias seguintes
})

</script>

