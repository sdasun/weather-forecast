<template>
  <div class="forecast">
    <div v-if="isLoading"><Spinner /></div>
    <div v-if="isLoaded" class="results">
      <div class="city">{{ cityName }}</div>
      <div class="country">{{ countryName }}</div>
      <div class="forecast-collection">
        <div v-for="(result, i) in results" :key="i" class="forecast-item">
          <div class="icon">
            <img
              :src="
                'http://openweathermap.org/img/w/' +
                result.weather[0].icon +
                '.png'
              "
            />
          </div>
          <div class="temp">
            <div class="label">Avg Temp:</div>
            <div class="value">
              {{ result.temp.avg | formatNumberDec(2) }}
              <span class="celsius">°C</span>
            </div>
          </div>
          <div class="weather-description">
            {{ result.weather[0].description | capitalize }}
          </div>
          <div class="date">{{ result.dt | unixToDate(result) }}</div>
        </div>
      </div>
    </div>
    <div v-if="isError" class="error-message">
      <div class="message">{{ errorMessage }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from "vue-property-decorator";
import { WeatherResponse, ForecastOneCallResponse, Daily } from "../models";
import Spinner from "./Spinner.vue";
import {
  getCountryNameByCode,
  unixToDate,
  formatNumberDec,
  capitalize,
} from "../utils";
@Component({
  components: {
    Spinner,
  },
  filters: {
    formatNumberDec,
    capitalize,
    unixToDate,
  },
})
export default class Forecast extends Vue {
  @Prop() private city!: string;
  @Prop() private days!: number;
  readonly apiKey = process.env.VUE_APP_WEATHER_API_KEY;
  readonly baseUrl = process.env.VUE_APP_BASE_URL;
  results?: Daily[];
  isLoading = false;
  isLoaded = false;
  isError = false;
  errorMessage = "";
  arrowCounter = -1;
  cityName? = "";
  countryName = "";
  created(): void {
    console.log("Created");
    this.$parent.$on("update", this.onCityChange);
  }
  @Watch("city")
  onCityChange(value: string): void {
    // Do not have access to the daily API (only paid), so using one-call api instead.
    // fetch(
    //   `${this.baseUrl}/data/2.5/forecast/daily?q=${value}&cnt=${this.days}&units=metric&appid=${this.apiKey}`
    // )
    //   .then((response) => response.json())
    //   .then((data: WeatherResponse) => {
    //     console.log(data);
    //     if (data.cod === 200) {
    //       this.results = data;
    //       this.results.countryName = getCountryNameByCode(
    //         data?.sys?.country as string
    //       );
    //       this.isLoading = false;
    //       this.isLoaded = true;
    //       console.log("success");
    //     } else {
    //       this.handleWeatherError(data);
    //       console.log("fail");
    //     }
    //   })
    //   .catch((error) => this.handleWeatherError(error));

    fetch(
      `${this.baseUrl}/data/2.5/weather?q=${value}&units=metric&appid=${this.apiKey}`
    )
      .then((response) => response.json())
      .then((response: WeatherResponse) => {
        if (response.cod === 200 && response.coord) {
          this.cityName = response.name;
          if (response.sys) {
            this.countryName = getCountryNameByCode(response.sys.country);
          }
          return fetch(
            `${this.baseUrl}/data/2.5/onecall?lat=${response.coord.lat}&lon=${response.coord.lon}&exclude=current,hourly,minutely,alerts&units=metric&appid=${this.apiKey}`
          );
        } else {
          throw response;
        }
      })
      .then((response) => response.json())
      .then((data: ForecastOneCallResponse) => {
        // remove first item (current weather)
        const daily = data.daily.slice(1);
        this.results = daily.map((item: Daily) => {
          item.temp.avg = (item.temp.min + item.temp.max) / 2;
          return item;
        });
        this.isLoading = false;
        this.isLoaded = true;
        console.log("success");
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
        this.errorMessage = JSON.stringify(error);
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
.forecast {
  display: flex;
  flex-direction: column;
  .results {
    .forecast-collection {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      justify-content: center;
      margin-top: 20px;
      font-size: 0.8rem;
      .forecast-item {
        width: 120px;
        font-size: 0.8rem;
      }
      @media (max-width: 992px) {
        flex-direction: column;
        align-items: center;
        .forecast-item:nth-child(odd) {
          background: #ccc;
          width: calc(100% - 40px);
          padding-bottom: 0.8rem;
        }
      }
    }
    justify-content: center;
    padding-top: 20px;
    display: flex;
    flex-direction: column;
    font-size: 0.8rem;
    .city {
      padding-bottom: 10px;
      font-size: 1.2rem;
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
      .label {
        padding-bottom: 5px;
      }
    }
    .weather-main {
      padding-bottom: 10px;
      font-size: 0.8rem;
    }
    .weather-description {
      font-size: 0.8rem;
      padding-bottom: 10px;
    }
  }
}
</style>
