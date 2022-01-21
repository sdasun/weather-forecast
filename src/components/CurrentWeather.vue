<template>
  <div>
    <div v-if="isLoading"><Spinner /></div>
    <div v-if="isLoaded" class="results">
      <div class="city" v-on:click="onCityClick()">{{ results.name }}</div>
      <div class="country">
        {{ results.sys.country | getCountryNameByCode }}
      </div>
      <div class="icon">
        <img
          :src="
            'http://openweathermap.org/img/w/' +
            results.weather[0].icon +
            '.png'
          "
        />
      </div>
      <div class="temp">
        {{ results.main.temp }} <span class="celsius">°C</span>
      </div>
      <div class="weather-main">{{ results.weather[0].main }}</div>
    </div>
    <Sunrise
      v-if="showSunrise"
      @closeSunrise="onCloseSunrise"
      :timezone="results.timezone"
      :sunrise="results.sys.sunrise"
      :sunset="results.sys.sunset"
      :city="results.name"
      :country="results.sys.country"
    />
    <div v-if="isError" class="error-message">
      <div class="message">{{ errorMessage }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from "vue-property-decorator";
import { WeatherResponse } from "../models";
import Spinner from "./Spinner.vue";
import Sunrise from "./Sunrise.vue";
import { getCountryNameByCode } from "../utils";
@Component({
  components: {
    Spinner,
    Sunrise,
  },
  filters: {
    getCountryNameByCode,
  },
})
export default class CurrentWeather extends Vue {
  @Prop() private city!: string;
  readonly apiKey = process.env.VUE_APP_WEATHER_API_KEY;
  readonly baseUrl = process.env.VUE_APP_BASE_URL;
  results?: WeatherResponse;
  isLoading = false;
  isLoaded = false;
  isError = false;
  errorMessage = "";
  arrowCounter = -1;
  showSunrise = false;

  onCloseSunrise(): void {
    this.showSunrise = false;
  }
  onCityClick(): void {
    this.showSunrise = true;
  }
  @Watch("city")
  onCityChange(value: string): void {
    fetch(
      `${this.baseUrl}/data/2.5/weather?q=${value}&units=metric&appid=${this.apiKey}`
    )
      .then((response) => response.json())
      .then((data: WeatherResponse) => {
        console.log(data);
        if (data.cod === 200) {
          this.results = data;
          this.results.countryName = getCountryNameByCode(
            data?.sys?.country as string
          );
          this.isLoading = false;
          this.isLoaded = true;
          console.log("success");
        } else {
          this.handleWeatherError(data);
          console.log("fail");
        }
      })
      .catch((error) => this.handleWeatherError(error));
    this.isError = false;
    this.isLoading = true;
    this.isLoaded = false;
  }

  private handleWeatherError(error: WeatherResponse): void {
    {
      if (error.cod === "404") {
        this.errorMessage = "City not found";
      } else if (error.message && typeof error.message === "string") {
        this.errorMessage = error.message;
      } else {
        this.errorMessage = "Something went wrong";
      }
      this.results = undefined;
      this.isError = true;
      this.isLoading = false;
      console.log(error);
    }
  }
}
</script>

<style lang="less" scoped>
.results {
  padding-top: 20px;
  display: flex;
  flex-direction: column;
  font-size: 1.2rem;
  .city {
    padding-bottom: 10px;
    font-size: 1.2rem;
    cursor: pointer;
  }
  .country {
    padding-bottom: 10px;
    font-size: 1rem;
  }
  .icon {
    padding-bottom: 10px;
  }
  .temp {
    padding-bottom: 10px;
  }
  .weather-main {
    padding-bottom: 10px;
  }
}
</style>
