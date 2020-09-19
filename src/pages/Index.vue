<template>
  <q-page class="flex flex-center">
    <h2>{{ location.name }}</h2>
    <h4 v-if="weather[0]">{{ weather[0].main }}</h4>
    <h4>{{ main.temp }}</h4>
    <p v-if="weather[0]">{{ weather[0].description }}</p>
  </q-page>
</template>

<script>
import axios from 'axios'

// TODO: move this to env var.
const apiKey = 'cdc06fab41045afce5f38fa0f21e9dee'
axios.interceptors.response.use((response) => {
  console.log('success')
  return {
    data: response.data,
    status: response.status,
    statusText: response.statusText,
  }
})

export default {
  name: 'PageIndex',
  data() {
    return {
      location: {
        lat: '',
        long: '',
        name: '',
      },
      weather: {},
      main: {},
    }
  },
  mounted() {
    this.getCurrentWeather()
  },
  methods: {
    getDeviceLocation() {
      return new Promise((resolve, reject) => {
        if ('geolocation' in window.navigator) {
          window.navigator.geolocation.getCurrentPosition((pos) => {
            const coords = pos.coords || ''
            if (coords && coords.latitude && coords.longitude) {
              this.location.lat = coords.latitude
              this.location.long = coords.longitude
            }
            resolve({ success: true })
          })
        } else {
          reject({ success: false })
        }
      })
    },
    async getCurrentWeather() {
      const locRes = await this.getDeviceLocation()
      if (locRes.success) {
        const url = `https://api.openweathermap.org/data/2.5/weather?lat=${this.location.lat}&lon=${this.location.long}&appid=${apiKey}&units=metric`
        const weatherRes = await axios.get(url)
        if (weatherRes.statusText === 'OK') {
          console.log('result', weatherRes)
          this.weather = weatherRes.data.weather
          this.main = weatherRes.data.main
          this.location.name = weatherRes.data.name
        } else {
          console.error('Fetch weather failed', weatherRes.status)
        }
      }
    },
  },
};
</script>
