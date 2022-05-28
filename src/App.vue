<template>
  <div id="app">
    <AppSidebar
        :officesByCities="officesByCities"
        :currentCountry="currentCountry"
        :currentCity="currentCity"
        :setCurrentOffice="setCurrentOffice"
        @setCountry="setCountry"
        @setCurrentCity="setCurrentCity"
    ></AppSidebar>
    <MapComponent
        :offices="offices"
        :currentCountry="currentCountry"
        :currentCity="currentCity"
        :currentOffice="currentOffice"
        :setCurrentOffice="setCurrentOffice"
        :setCurrentCity="setCurrentCity"
    ></MapComponent>
  </div>
</template>

<script>
import AppSidebar from '@/components/AppSidebar'
import MapComponent from '@/components/MapComponent'

export default {
  name: 'App',
  components: {
    AppSidebar,
    MapComponent
  },

  // В состоянии храним массив всех офисов, выбранную страну, город и конкретный офис
  data() {
    return {
      offices: null,
      currentCountry: 'россия',
      currentCity: null,
      currentOffice: null,
    }
  },

  // В компонент сайдбара отдаем массив офисов только для выбранной страны
  computed: {
    officesByCities() {
      if (this.offices) {
        const result = this.offices?.countries.filter( item => {
          return item.name.toLowerCase() === this.currentCountry
        })
        return result[0].cities
      } else {
        return []
      }
    }
  },
  methods: {
    // Сеттер текущей страны, доступный для дочерних компонентов
    setCountry(value) {
      this.currentCountry = value
      this.currentCity = null
    },

    // Сеттер текущего города, доступный для дочерних компонентов
    setCurrentCity(value) {
      this.currentCity === value ? this.currentCity = null : this.currentCity = value
    },

    // Сеттер текущего офиса, доступный для дочерних компонентов
    setCurrentOffice(value) {
      this.currentOffice = value
    }
  },

  // Наблюдаем за состоянием текущего офиса, синхронизируем текущий город с тем,
  // что относится к выбранному офису
  watch: {
    currentOffice( newValue ) {
      this.currentCity = newValue.city
    }
  },

  // Парсим данные в формате JSON и записываем в состояние
  beforeCreate() {
    const requestURL = 'https://raw.githubusercontent.com/VictorNov/vue-map/master/src/data.json'
    const request = new XMLHttpRequest()
    request.open('GET', requestURL)
    request.responseType = 'json'
    request.send()
    request.onload = () => {
      this.offices = request.response
    }
  }
}
</script>

<style lang="scss">
#app {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: stretch;

  @media screen and (max-width: 768px) {
    flex-direction: column-reverse;
  }
}
</style>
