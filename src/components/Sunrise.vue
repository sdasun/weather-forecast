<template>
  <div>
    <div class="backdrop" v-on:click="onClose()"></div>
    <div class="dialog-box">
      <div class="dialog-content">
        <div class="dialog-header">
          <div class="close-button" v-on:click="onClose">
            <svg
              aria-hidden="true"
              focusable="false"
              data-prefix="fas"
              data-icon="times"
              class="svg-inline--fa fa-times fa-w-11"
              role="img"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 352 512"
            >
              <path
                fill="currentColor"
                d="M242.72 256l100.07-100.07c12.28-12.28 12.28-32.19 0-44.48l-22.24-22.24c-12.28-12.28-32.19-12.28-44.48 0L176 189.28 75.93 89.21c-12.28-12.28-32.19-12.28-44.48 0L9.21 111.45c-12.28 12.28-12.28 32.19 0 44.48L109.28 256 9.21 356.07c-12.28 12.28-12.28 32.19 0 44.48l22.24 22.24c12.28 12.28 32.2 12.28 44.48 0L176 322.72l100.07 100.07c12.28 12.28 32.2 12.28 44.48 0l22.24-22.24c12.28-12.28 12.28-32.19 0-44.48L242.72 256z"
              ></path>
            </svg>
          </div>
          <h3 class="title">Sunrise &amp; Sunset</h3>
          <div class="sunrise-set-date">
            {{ subtitle }}
          </div>
          <!-- close button -->
        </div>
        <div class="dialog-body">
          <img class="sun-logo" src="@/assets/images/sun.svg" />
          <div class="rise-n-set">
            <div class="sunrise">
              <div class="sunrise-value">{{ sunriseFormated }}</div>
              <div class="sunrise-label">Sunrise</div>
            </div>
            <div class="sunrise">
              <div class="sunrise-value">{{ sunsetFormated }}</div>
              <div class="sunrise-label">Sunset</div>
            </div>
          </div>
          <div>
            <div class="day-n-night">
              <div class="info">
                Day Length: <b>{{ dayLength }}</b>
              </div>
              <div class="info">
                Night Length: <b>{{ nightLength }}</b>
              </div>
            </div>
          </div>
        </div>
        <div class="dialog-footer">
          <i>
            Please note, the sunrise and sunset times are based on the location
            of the city you selected. This city has
            {{ timezone | secondsToTime }} of offset from UTC.
          </i>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from "vue-property-decorator";
import {
  unixToDateTime,
  unixToDateWithTimezone,
  unixToTime,
  secondsToTime,
} from "../utils";
@Component({
  filters: {
    secondsToTime,
  },
})
export default class Spinner extends Vue {
  @Prop() private sunrise!: number;
  @Prop() private sunset!: number;
  @Prop() private timezone!: number;
  @Prop() private city!: string;
  @Prop() private country!: string;

  private onClose() {
    this.$emit("closeSunrise");
  }

  get sunriseFormated(): string {
    const riseDate = unixToDateWithTimezone(this.sunrise, this.timezone);
    const setDate = unixToDateWithTimezone(this.sunset, this.timezone);
    if (riseDate === setDate) {
      return unixToTime(this.sunrise, this.timezone);
    }
    return unixToDateTime(this.sunrise, this.timezone);
  }

  get sunsetFormated(): string {
    const riseDate = unixToDateWithTimezone(this.sunrise, this.timezone);
    const setDate = unixToDateWithTimezone(this.sunset, this.timezone);
    if (riseDate === setDate) {
      return unixToTime(this.sunset, this.timezone);
    }
    return unixToDateTime(this.sunset, this.timezone);
  }
  isRiseNSetOnSameDay(): boolean {
    const riseDate = unixToDateWithTimezone(this.sunrise, this.timezone);
    const setDate = unixToDateWithTimezone(this.sunset, this.timezone);
    return riseDate === setDate;
  }
  get subtitle(): string | undefined {
    const riseDate = unixToDateWithTimezone(this.sunrise, this.timezone);
    const setDate = unixToDateWithTimezone(this.sunset, this.timezone);
    if (riseDate === setDate) {
      return `of ${this.city}, ${this.country} on ${riseDate}`;
    }
    return `of ${this.city}, ${this.country} `;
  }

  get dayLength(): string {
    return secondsToTime(this.sunset - this.sunrise, false);
  }

  get nightLength(): string {
    return secondsToTime(86400 - this.sunset + this.sunrise, false);
  }
}
</script>

<style lang="less" scoped>
.title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 0.2rem;
  margin-top: 0;
}
.sunrise-set-date {
  font-size: 1rem;
  font-weight: bold;
  color: #999;
  margin-bottom: 0.5rem;
}
.sun-logo {
  width: 100px;
  height: 100px;
  margin: 0 auto;
}
.dialog-box {
  display: flex;
  justify-content: center;
  padding: 20px;
  width: 400px;
  min-height: 300px;
  background-color: #fff;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1000;
  border-radius: 5px;
  // box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  box-shadow: rgba(37, 38, 39, 0.03) 0px -1px 2px 0px,
    rgba(37, 38, 39, 0.04) 0px 3px 2px -2px,
    rgba(37, 38, 39, 0.04) 0px 7px 5px -2px,
    rgba(37, 38, 39, 0.05) 0px 12px 10px -2px,
    rgba(37, 38, 39, 0.06) 0px 22px 18px -2px,
    rgba(37, 38, 39, 0.07) 0px 41px 33px -2px,
    rgba(37, 38, 39, 0.08) 0px 100px 80px -2px;
  @media (max-width: 480px) {
    width: calc(100% - 40px);
    padding: 10px;
  }
  .close-button {
    position: absolute;
    top: 2px;
    right: 4px;
    font-size: 1.5rem;
    color: #999;
    cursor: pointer;
    width: 16px;
    padding: 5px;
  }
}
.day-n-night .info {
  font-size: 0.9rem;
  color: #999;
  margin-bottom: 0.5rem;
  padding: 0 0.5rem;
}
.rise-n-set,
.day-n-night {
  display: flex;
  justify-content: center;
}
.sunrise {
  margin-top: 10px;
}
.sunrise,
.sunset {
  padding: 0 10px;
  margin-bottom: 10px;
}

.sunrise-value,
.sunset-value {
  font-size: 1.2rem;
  font-weight: bold;
}

.sunrise-label,
.sunset-label {
  font-size: 0.8rem;
  font-weight: normal;
}

.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 999;
}
</style>
