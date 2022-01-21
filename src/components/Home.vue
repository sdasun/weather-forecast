<template>
  <div class="home">
    <h1>{{ msg }}</h1>

    <SearchText @input="onSearch" />
    <div class="tab">
      <div
        class="tab-item-title"
        v-on:click="updateTab(i)"
        v-bind:class="{ active: tab.isActive }"
        v-for="(tab, i) in tabs"
        :key="i"
      >
        {{ tab.title }}
      </div>
    </div>
    <CurrentWeather
      ref="currentWeather"
      :city="searchText"
      v-show="tabs[0].isActive"
    />
    <Forecast
      ref="forecst"
      :city="searchText"
      v-show="tabs[1].isActive"
      :days="7"
    />
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import SearchText from "./SearchText.vue";
import CurrentWeather from "./CurrentWeather.vue";
import Forecast from "./Forecast.vue";
@Component({
  components: {
    SearchText,
    CurrentWeather,
    Forecast,
  },
})
export default class Home extends Vue {
  @Prop() private msg!: string;
  tabs = [
    { title: "Current", isActive: true, component: CurrentWeather },
    { title: "7 Day Forecast", isActive: false, component: Forecast },
  ];
  private msg2 = process.env.VUE_APP_WEATHER_API_KEY;
  searchText = "";
  onSearch(value: string): void {
    this.searchText = value;
    console.log("onSearch", event);
  }
  updateTab(index: number): void {
    this.tabs.forEach((t) => {
      t.isActive = false;
    });
    Vue.set(this.tabs, index, {
      ...this.tabs[index],
      isActive: true,
    });
  }
}
</script>

<style scoped lang="less">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.tab {
  margin-top: 40px;
  .tab-item-title {
    display: inline-block;
    padding: 10px;
    cursor: pointer;
    text-transform: uppercase;
    &:hover {
      border-bottom: 2px solid #ccc;
    }
    &.active {
      background-color: #ccc;
      border-bottom: 2px solid #4283b9;
    }
  }
}
</style>
