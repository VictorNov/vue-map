<template>
  <div class="map__container" id="map">
  </div>
</template>

<script>
// Плагин для работы с Yandex Maps API
import { loadYmap } from 'vue-yandex-maps'

export default {
  name: 'MapComponent',
  props: {
    offices: Object,
    currentCountry: String,
    currentCity: String,
    currentOffice: Object,
    setCurrentOffice: Function,
    setCurrentCity: Function,
  },

  // В состоянии храним объект карты, коллекцию объектов, шаблоны меток и балуна
  data() {
    return {
      myMap: {},
      collection: {
        type: "FeatureCollection",
        features: [],
      },
      objectManager: null,
      settings: {
        apiKey: '2bbafa06-90bb-4c4e-8054-cd17ce4585c5',
        lang: 'ru_RU',
        coordorder: 'latlong',
        enterprise: false,
        version: '2.1'
      },
      myBalloonLayout: null,
      myCluster: null,
      myPlacemark: null,
      myPlacemarkActive: null,
    }
  },
  methods: {
    // Инициализация карты, функция получает объект ymaps, загружаемый асинхронно из vue-yandex-maps
    init( ymaps ) {
      // Создание экземпляра карты
      this.myMap = new ymaps.Map( 'map', {
        center: [ 55.76, 37.64 ],
        zoom: 7,
        controls: [],
      }, {
        yandexMapDisablePoiInteractivity: true,
      } )

      // Создание менеджера объектов
      this.objectManager = new ymaps.ObjectManager( {
        clusterize: true,
        gridSize: 64,
      } )

      // Создаем кастомный шаблон кластера
      this.myCluster = ymaps.templateLayoutFactory.createClass(
          '<div class="map__cluster">{{ properties.geoObjects.length }}</div>')

      // Создаем кастомный шаблон метки
      this.myPlacemark = ymaps.templateLayoutFactory.createClass(
          '<div class="map__placemark"></div>')

      // Создаем кастомный шаблон активной метки
      this.myPlacemarkActive = ymaps.templateLayoutFactory.createClass(
          '<div class="map__placemark active"></div>')

      // Создаем кастомный шаблон балуна
      this.myBalloonLayout = ymaps.templateLayoutFactory.createClass(`
        <div class="map__balloon">
          <button type="button" class="map__balloon-close"></button>
          <div class="map__balloon-arrow"></div>
          <div class="map__balloon-inner">
            $[[options.contentLayout observeSize]]
          </div>
        </div>`, {
          // Строит экземпляр макета на основе шаблона и добавляет его в родительский HTML-элемент.
          build: function () {
            this.constructor.superclass.build.call(this);
            this.element = document.querySelector('.map__balloon').parentNode
            this.applyElementOffset();

            // Слушаем клик по кнопке закрытия
            this.element.querySelector('.map__balloon-close').addEventListener('click', this.onCloseClick.bind(this))
          },

          //Сдвигаем балун, чтобы "хвостик" указывал на точку привязки.
          applyElementOffset: function () {
            this.element.querySelector('.map__balloon').style.left = '-65px'
            this.element.querySelector('.map__balloon').style.top = `-192px`
          },

          //Закрывает балун при клике на крестик, кидая событие "userclose" на макете.
          onCloseClick: function (e) {
            e.preventDefault();
            this.events.fire('userclose');
          },

          // Удаляет содержимое макета из DOM.
          clear: function () {
            this.element.querySelector('.map__balloon-close').removeEventListener('click', this.onCloseClick);
            this.constructor.superclass.clear.call(this);
          },
        })

      // Настраиваем внешний вид иконок кластеров
      this.objectManager.clusters.options.set( {
        clusterIconLayout: this.myCluster,
        clusterIconShape: {
          type: 'Circle',
          coordinates: [ 0, 0 ],
          radius: 19
        },
      } )

      // Настраиваем внешний вид иконок объектов
      this.objectManager.objects.options.set({
        hideIconOnBalloonOpen: false,
        iconLayout: this.myPlacemark,
        iconShape: {
          type: 'Circle',
          coordinates: [ 0, 0 ],
          radius: 12
        },
        balloonLayout: this.myBalloonLayout,
        balloonPanelMaxMapArea: 0
      } )

      // Создаем коллекцию объектов
      this.populateCollection()

      // Добавляем коллекцию в менеджер объектов
      this.objectManager.add( this.collection )

      // Менеджер объектов связываем с картой
      this.myMap.geoObjects.add( this.objectManager )

      // Применяем фильтр по стране по умолчанию
      this.filterCollectionByCountry( this.currentCountry )

      // Слушаем клики по иконкам и закрытие балуна
      this.myMap.balloon.events.add('close', this.onBalloonClose.bind(this, ymaps))
      this.objectManager.objects.events.add( 'click', this.onObjectClick.bind( this ) )
      this.objectManager.clusters.events.add('click', this.onClusterClick.bind(this))
    },

    // Функция преобразует полученный массив данных в формат, необходимый для ymaps.objectManager
    populateCollection() {
      this.offices.countries.forEach( country => {
        country.cities.forEach( city => {
          city.offices.forEach( office => {
            this.collection.features.push( {
              type: "feature",
              id: office.id,
              geometry: {
                type: "Point",
                coordinates: office.coordinates
              },
              properties: {
                balloonContentHeader: `
                  <p class="map__balloon-header">${office.name}</p>
                `,
                balloonContent: `
                  <p class="map__balloon-manager">${office.manager}</p>
                  <p class="map__balloon-phones">${office.phones.map( ( phone ) => {
                  return `<a href="tel:${phone}">${phone}</a>`
                } ).join( '' )}
                  </p>
                  <p class="map__balloon-email">
                    <a href="mailto:${office.email}">${office.email}</a>
                  </p>
                `,
                country: country.name,
                city: city.name,
              },
            } )
          } )
        } )
      } )
    },

    // Функция фильтрует коллекцию объектов в соответствии с выбранной страной и фокусирует карту на
    // всех полученных точках
    filterCollectionByCountry( country ) {
      this.objectManager.setFilter( object => {
        return object.properties.country.toLowerCase() === country.toLowerCase()
      })
      this.myMap.setBounds(
        this.myMap.geoObjects.getBounds(), {
          checkZoomRange: true,
          duration: 700,
          timingFunction: 'ease-in-out'
        }
      )
    },

    // Функция вызывается при раскрытии элемента аккордеона
    // Отфильтровывает объекты по соответствующему городу,
    // устанавливает границы видимой области так что помещаются все иконки
    // Возвращает остальные иконки на карту
    filterCollectionByCity( city ) {
      if ( city ) {
        // Показываем точки только текущего города
        this.objectManager.setFilter( object => {
          return object.properties.city.toLowerCase() === city.toLowerCase()
        } )

        // Фокусируемся на них
        this.myMap.setBounds( this.myMap.geoObjects.getBounds(), {
          checkZoomRange: true,
          duration: 700,
          timingFunction: 'ease-in-out'
        } )

        // Возвращаем остальные точки на карту
        this.objectManager.setFilter( object => {
          return object.properties.country.toLowerCase() === this.currentCountry.toLowerCase()
        } )
      } else {
        // При закрытии аккордеона снова показываем все точки на карте
        this.myMap.setBounds( this.myMap.geoObjects.getBounds(), {
          checkZoomRange: true,
          duration: 700,
          timingFunction: 'ease-in-out'
        } )
      }
    },

    // При клике на объекте получаем id и сам объект
    // Записываем id и название города в состояние компонента App
    onObjectClick( e ) {
      const objectId = e.get( 'objectId' )
      const object = this.objectManager.objects.getById( objectId )

      this.setCurrentOffice( {
        id: objectId,
        city: object.properties.city
      } )
    },

    // Фокусируемся на выбранном объекте
    focusOnObject( { id } ) {
      // Получаем объект по id
      const object = this.objectManager.objects.getById( id )

      // Устанавливаем центр карты
      this.myMap.setCenter( object.geometry.coordinates, 16 )

      // Раскрываем балун
      this.objectManager.objects.balloon.open( id )

      // Меняем стиль метки на активный
      this.setActivePlacemark(id)
    },

    // Функция принимает id объекта и применяет стиль активной иконке
    // того объекта, балун которого мы раскрыли
    // Если на вход передать false - все иконки изменят стиль на неактивный
    setActivePlacemark( id ) {
      this.objectManager.objects.each( object => {
        if ( object.id === id ) {
          this.objectManager.objects.setObjectOptions( object.id, {
            iconLayout: this.myPlacemarkActive
          } )
        } else {
          this.objectManager.objects.setObjectOptions( object.id, {
            iconLayout: this.myPlacemark
          } )
        }
      } )
    },

    // При клике по кластеру переписываем currentCity в состоянии компонента App
    onClusterClick(e) {
      const cluster = this.objectManager.clusters.getById(e.get('objectId'))
      this.setCurrentCity(cluster.features[0].properties.city)
    },

    // При закрытии балуна меняем иконку объекта на неактивную
    onBalloonClose() {
      this.setActivePlacemark(false)
    }
  },
  // При монтировании настраиваем API и передаем объект ymaps функции this.init()
  async mounted() {
    await loadYmap({ ...this.settings });

    // eslint-disable-next-line
    ymaps.ready(this.init.bind(this, ymaps))
  },
  watch: {
    // Наблюдаем изменение текущей страны, фильтруем объекты на карте
    currentCountry( newValue ) {
      this.filterCollectionByCountry(newValue)
    },
    // Наблюдаем изменение текущего города и фокусируем объекты на карте
    currentCity( newValue ) {
      this.filterCollectionByCity(newValue)
    },
    // Наблюдаем изменение текущего офиса и фокусируем карту на конкретном офисе
    currentOffice( newValue ) {
      this.focusOnObject(newValue)
    },
  },
}
</script>

<style lang="scss">
.map__container {
  max-height: 100vh;
  flex-grow: 1;

  // Make map grayscale
  canvas {
    filter: grayscale(1);
  }

  ymaps[class$='-ground-pane'] {
    filter: grayscale(1);
  }

  // Hide Copyrights
  ymaps[class$='-copyrights-pane'] {
    display: none;
  }
}

// Custom placemark icon
.map__placemark {
  position: relative;
  width: 24px;
  height: 24px;
  border: 2px solid #FFFFFF;
  background-color: #1e355d;
  border-radius: 50px;
  left: -12px;
  top: -12px;

  &.active {
    -webkit-box-shadow: 0 0 29px rgba(3, 51, 134, .5);
    -moz-box-shadow: 0 0 29px rgba(3, 51, 134, .5);
    box-shadow: 0 0 29px rgba(3, 51, 134, .5);
  }
}

// Custom cluster icon
.map__cluster {
  position: relative;
  width: 38px;
  height: 38px;
  border: 2px solid #FFFFFF;
  background-color: #1e355d;
  color: #FFFFFF;
  border-radius: 50px;
  left: -19px;
  top: -19px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 13px;
}

// Custom Balloon
.map__balloon {
  position: relative;
  display: block;
  background-color: #253b62;
  color: #FFFFFF;
  width: 375px;
  height: 180px;
  padding: 20px 30px 45px;
}

.map__balloon-close {
  position: absolute;
  right: 10px;
  top: 10px;
  width: 12px;
  height: 12px;
  cursor: pointer;
  background: none;

  &::before,
  &::after {
    content: '';
    display: block;
    width: 14px;
    height: 2px;
    background-color: #fff;
    position: absolute;
    left: 50%;
    top: 50%;
  }
  
  &:hover::before,
  &:hover::after {
    background-color: #ff9e00;
  }
  
  &::before {
    transform: translate(-50%, -50%) rotate(45deg);
  }
  
  &::after {
    transform: translate(-50%, -50%) rotate(-45deg);
  }
}

.map__balloon-arrow {
  position: absolute;
  width: 10px;
  height: 10px;
  background-color: #253b62;
  bottom: 0;
  left: 60px;
  transform: translateY(50%) rotate(45deg);
}

.map__balloon-inner ymaps {
  overflow: visible !important;
}

.map__balloon-header {
  color: #ff9e00;
  font-weight: bold;
  font-size: 14px;
  line-height: 14px;
  margin-bottom: 25px;
}

.map__balloon-manager,
.map__balloon-phones {
  color: #FFFFFF;
  font-weight: normal;
  font-size: 14px;
  line-height: 14px;
  margin-bottom: 20px;

  a {
    text-decoration: none;
    color: #FFFFFF;
    margin-right: 20px;

    &:hover {
      color: #ff9e00;
      text-decoration: underline;
    }
  }
}

.map__balloon-email {
  a {
    text-decoration: none;
    color: #32b3e9;
    font-size: 16px;

    &:hover {
      color: #ff9e00;
      text-decoration: underline;
    }
  }
}
</style>