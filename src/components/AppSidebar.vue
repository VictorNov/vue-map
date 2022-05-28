<template>
  <aside class="sidebar">
    <div class="sidebar__countries">
      <button
          :class="['sidebar__countries-button', {'button-active': currentCountry === 'россия'}]"
          @click="$emit('setCountry', 'россия')"
      >
        россия
      </button>
      <button
          :class="['sidebar__countries-button', {'button-active': currentCountry === 'беларусь'}]"
          @click="$emit('setCountry', 'беларусь')"
      >беларусь</button>
    </div>
    <div class="sidebar__accordion">
      <AccordionItem
          v-for="city in officesByCities"
          :city="city"
          :currentCity="currentCity"
          :setCurrentOffice="setCurrentOffice"
          :key="city.name"
          @setCurrentCity="$emit('setCurrentCity', $event)"
      />
    </div>
  </aside>
</template>

<script>
import AccordionItem from '@/components/AccordionItem'

export default {
  name: 'AppSidebar',
  props: {
    officesByCities: Array,
    currentCountry: String,
    currentCity: String,
    setCurrentOffice: Function
  },
  components: {
    AccordionItem,
  },
}
</script>

<style lang="scss" scoped>
.sidebar {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  width: 400px;
  height: 100vh;

  background-color: #FAFAFA;
  -webkit-box-shadow: 0 0 8px 2px rgba(0, 0, 0, .17);
  -moz-box-shadow: 0 0 8px 2px rgba(0, 0, 0, .17);
  box-shadow: 0 0 8px 2px rgba(0, 0, 0, .17);
  z-index: 1;

  @media screen and (max-width: 768px) {
    max-height: 50%;
    width: 100%;
  }
}

.sidebar__countries {
  height: 34px;
  width: 100%;
  display: flex;
  flex-direction: row;
  align-items: stretch;
  flex-shrink: 0;
}

.sidebar__countries-button {
  flex-grow: 1;
  border: 1px solid #929292;

  background-color: #FFFFFF;
  color: #929292;

  font-family: 'Helvetica Neue Cyr', sans-serif;
  font-size: 12px;
  font-weight: bold;
  text-transform: capitalize;

  -webkit-transition: all .3s;
  -moz-transition: all .3s;
  -ms-transition: all .3s;
  -o-transition: all .3s;
  transition: all .3s;

  &:not(.button-active):hover {
    background-color: #ffce83;
    color: #000;
  }
}

.button-active {
  background-color: #FF9E00;
  color: #FFFFFF;
  border: none;
  cursor: default;
}

.sidebar__accordion {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: stretch;
  overflow-y: auto;
  &::-webkit-scrollbar {
    display: none;
  }
}
</style>