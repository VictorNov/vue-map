<template>
  <div :class="['accordion__item', {'accordion-open': currentCity === city.name}]">
    <button
        class="accordion__header"
        @click="$emit('setCurrentCity', city.name)"
    >
      <span>{{city.name}}</span>
      <span class="accordion__icon" />
    </button>
    <div class="accordion__content">
      <button class="accordion__content-card"
              v-for="office in city.offices"
              :key="office.id"
              @click="setCurrentOffice({id: office.id, city: city.name})"
      >
        <span class="accordion__content-card-name">{{office.name}}</span>
        <span class="accordion__content-card-manager">{{office.manager}}</span>
        <span class="accordion__content-card-phones">
          <a
              v-for="(phone, i) in office.phones"
              :key="i"
              :href="'tel:' + phone">
            {{phone}}
          </a>
        </span>
        <span class="accordion__content-card-email">
          <a :href="'mailto:' + office.email">{{office.email}}</a>
        </span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AccordionItem',
  props: {
    city: Object,
    currentCity: String,
    setCurrentOffice: Function,
  },
}
</script>

<style lang="scss" scoped>
.accordion__item {
  border-bottom: 1px solid #EDEDED;
  font-size: 14px;
}

.accordion__header {
  height: 76px;
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding-left: 25px;
  padding-right: 40px;
  color: #00325E;
  font-weight: bold;
  border: none;
  background: none;

  -webkit-transition: all .3s;
  -moz-transition: all .3s;
  -ms-transition: all .3s;
  -o-transition: all .3s;
  transition: all .3s;
  
  &:hover {
    background-color: #fff;
  }
}

.accordion__icon {
  display: block;
  width: 8px;
  height: 8px;
  border-bottom: 2px solid #00325E;
  border-left: 2px solid #00325E;
  border-top: none;
  border-right: none;
  -ms-transform: translateY(-4px) rotate(-45deg);
  transform: translateY(-4px) rotate(-45deg);

  -webkit-transition: all .3s;
  -moz-transition: all .3s;
  -ms-transition: all .3s;
  -o-transition: all .3s;
  transition: all .3s;
}

.accordion-open{
  .accordion__header {
    color: #FF9E00;
  }

  .accordion__icon {
    -ms-transform: translateY(0px) rotate(135deg);
    transform: translateY(0px) rotate(135deg);
    border-color: #FF9E00;
  }

  .accordion__content {
    height: max-content;
    display: block;
  }
}

.accordion__content {
  height: 0;
  overflow: hidden;
  display: none;
  -webkit-transition: all .5s ease-in-out;
  -moz-transition: all .5s ease-in-out;
  -ms-transition: all .5s ease-in-out;
  -o-transition: all .5s ease-in-out;
  transition: all .5s ease-in-out;
}

.accordion__content-card {
  padding: 10px 25px 13px;
  text-align: start;
  width: 100%;
  background: none;

  -webkit-transition: all .3s;
  -moz-transition: all .3s;
  -ms-transition: all .3s;
  -o-transition: all .3s;
  transition: all .3s;

  span {
    display: block;
    margin-bottom: 22px;
  }

  &:hover {
    background-color: #fff;
  }
}

.accordion__content-card-name {
  font-weight: bold;
  color: #00325E;
  margin-bottom: 25px;
}

.accordion__content-card-phones {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  flex-wrap: wrap;
  row-gap: 10px;

  a {
    text-decoration: none;
    color: #000000;
    margin-right: 11px;

    &:hover {
      text-decoration: underline;
    }
  }
}

.accordion__content-card-email a {
  color: #32B3E9;

  &:hover {
    color: #00325E;
  }
}
</style>