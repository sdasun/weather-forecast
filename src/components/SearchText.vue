<template>
  <div class="searchtext">
    <div class="searchtext-input">
      <input
        type="text"
        placeholder="Enter a city. Example: London, UK"
        alt="Here you enter a city to get weather and forcast. Example: London, UK"
        @input="onChange"
        v-model="search"
        @keydown.down="onArrowDown"
        @keydown.up="onArrowUp"
        @keydown.enter="onEnter"
      />
      <button type="button" @click="onSearch()">
        <svg
          aria-hidden="true"
          focusable="false"
          data-prefix="fas"
          data-icon="search"
          class="svg-inline--fa fa-search fa-w-16"
          role="img"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 512 512"
        >
          <path
            d="M505 442.7L405.3 343c-4.5-4.5-10.6-7-17-7H372c27.6-35.3 44-79.7 44-128C416 93.1 322.9 0 208 0S0 93.1 0 208s93.1 208 208 208c48.3 0 92.7-16.4 128-44v16.3c0 6.4 2.5 12.5 7 17l99.7 99.7c9.4 9.4 24.6 9.4 33.9 0l28.3-28.3c9.4-9.4 9.4-24.6.1-34zM208 336c-70.7 0-128-57.2-128-128 0-70.7 57.2-128 128-128 70.7 0 128 57.2 128 128 0 70.7-57.2 128-128 128z"
          ></path>
        </svg>
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
@Component({})
export default class SearchText extends Vue {
  search = "";
  onSearch(): void {
    if (this.search && this.search.length > 1) {
      this.$emit("input", this.search);
    }
  }
  onEnter(): void {
    this.onSearch();
  }
}
</script>

<style lang="less">
.searchtext {
  display: flex;
  justify-content: center;
  position: relative;
  height: 28px;
  .searchtext-input {
    min-width: 600px;
    position: relative;
    @media (max-width: 768px) {
      min-width: 400px;
    }
    @media (max-width: 576px) {
      min-width: 300px;
    }
    @media (max-width: 480px) {
      min-width: calc(100% - 40px);
    }
    input {
      width: 100%;
      height: 28px;
      border: none;
      box-shadow: 0px 1px 1px 1px #ccc;
      &:focus {
        outline: none;
        box-shadow: 0px 1px 1px 1px #999;
      }
      ::placeholder {
        color: #ccc;
      }
      padding: 5px;
      box-sizing: border-box;
    }
    button {
      border: none;
      position: absolute;
      right: 1px;
      top: 1px;
      height: 24px;
      overflow: hidden;
      background: #fff;
      cursor: pointer;
      outline: none;
    }
  }
}

.searchtext-results {
  padding: 0;
  margin: 0;
  border: 1px solid #eeeeee;
  height: 120px;
  overflow: auto;
}

.searchtext-result {
  list-style: none;
  text-align: left;
  padding: 4px 2px;
  cursor: pointer;
}

.searchtext-result.is-active,
.searchtext-result:hover {
  background-color: #4aae9b;
  color: white;
}

.svg-inline--fa.fa-search {
  fill: #ccc;
  width: 20px;
  padding: 2px 0 0 0;
  &:hover {
    fill: #4aae9b;
  }
}
</style>
