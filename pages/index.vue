<template>
  <div class="container">
    <div class="top">
      <h1 class="title">OS: iOS | Platform: iPad | Country: {{ country }}</h1>
      <h1 class="title">Size: {{ totalSize }}</h1><br>
      <input class="search-query" type="text" placeholder="Search App Name" v-model="searchQuery" v-on:keyup.enter="appStoreSearch">

      <!-- The Search List -->
      <ul v-if="this.searchResult.length != 0" class="query-list" ref="listSearch">
        <li v-for="(app, i) in searchResult" :key="app.bundleId" class="app-list" @click="selectApp(i)">
          <img class="app-icon" :src="app.artworkUrl100">
          <div style="width:60%;float:right;" :style="{ height: searchAppHeight }">
            <p class="app-name">{{ app.trackName }}</p>
          </div>
        </li>
      </ul>
      <br><br>
      
      <!-- The List of Selected Apps -->
      <ul class="selected-list" ref="listApp">
        <li v-for="app in selectedApps" :key="app.bundleId" class="app-list" style="cursor:not-allowed;" @click="removeApp(app.bundleId)">
          <img class="app-icon" :src="app.icon">
          <div style="width:60%;float:right;" :style="{ height: searchAppHeight }">
            <p class="app-name">{{ app.name }}</p>
            <p class="app-name">{{ sizeOf(app.sizeBytes) }}</p>
          </div>
        </li>
      </ul>
    </div>
    
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

import axios from 'axios'

export default Vue.extend({
  data: () => ({
    /// Lookup Tables
    iosPlatforms: {
      "iPhone": "software",
      "iPad": "iPadSoftware",
    },

    /// Settings
    country: "",

    /// DOM
    searchQuery: "",
    searchResult: [],
    selectedApps: [],

    /// DOM style
    searchAppHeight: "100px"
  }),
  mounted() {
    this.country = (navigator.language.split('-').at(-1) == undefined) ? "US" : (navigator.language.split('-').at(-1)) as string;
  },
  computed: {
    totalSize() {
      let size = 0;
      for (var i = 0; i < this.selectedApps.length; i++) {
        // @ts-ignore
        size += parseInt(this.selectedApps[i].sizeBytes);
      }

      if (size == 0) { return "0.00 B"; }
      var e = Math.floor(Math.log(size) / Math.log(1024));
      return (size/Math.pow(1024, e)).toFixed(2)+' '+' KMGTP'.charAt(e)+'B';
    },
  },
  watch: {
    searchQuery: function(newVal, oldVal) {
      if (oldVal != "" && newVal == "") {
        this.searchResult = [];
      }
    }
  },
  methods: {
    getSearchAppHeight() {
      // console.log(this.$refs['listSearch']);
      return "40px";
    },
    sizeOf(bytes: number) {
      if (bytes == 0) { return "0.00 B"; }
      var e = Math.floor(Math.log(bytes) / Math.log(1024));
      return (bytes/Math.pow(1024, e)).toFixed(2)+' '+' KMGTP'.charAt(e)+'B';
    },
    /// Get search results
    appStoreSearch() {
      // https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/iTuneSearchAPI/Searching.html
      axios({
        method: 'get',
        // entity: software, iPadSoftware, macSoftware
        // url: 'http://itunes.apple.com/search?entity=software&term=' + this.searchQuery,
        url: 'http://itunes.apple.com/search?entity=iPadSoftware&country=' + this.country + '&term=' + this.searchQuery,
      })
      .then( (r: any) => {
        this.searchResult = r.data.results;
        // console.log(this.searchResult);

        this.$nextTick( () => this.getSearchAppHeight() );
      })
      .catch( (error: any) => {
        console.log(error.response.data);
      });
    },
    selectApp(id: number) {
      // @ts-ignore
      let selectedApp = {
        // @ts-ignore
        bundleId: this.searchResult[id].bundleId,
        // @ts-ignore
        icon: this.searchResult[id].artworkUrl512,
        // @ts-ignore
        name: this.searchResult[id].trackName,
        // @ts-ignore
        sizeBytes: this.searchResult[id].fileSizeBytes,
      };
      // console.log(selectedApp);

      this.searchQuery = "";
      this.searchResult = [];

      // @ts-ignore
      this.selectedApps.push(selectedApp);
    },
    removeApp(bid: string) {
      let name = "";
      this.selectedApps.forEach( (app) => {
        // @ts-ignore
        if (app.bundleId == bid) {
          // @ts-ignore
          name = app.name;
          return;
        }
      });

      if (confirm("You wanna remove " + name + "?")) {
        /// Remove the app from the list
        for (var i = 0; i < this.selectedApps.length; i++) {
          // @ts-ignore
          if (this.selectedApps[i].bundleId == bid) {
            this.selectedApps.splice(i, 1);
          }
        }
      }
    }
  }
})
</script>

<style lang="scss">
body {
  padding: 0;
  margin: 0;
}
html {
  font-family:
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  font-size: 16px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
}

.container {
  .top {
    width: 70%;
    max-width: 600px;

    position: absolute;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);

    .title {
      text-align: center;
    }

    .search-query {
      width: 100%;

      // position: absolute;
      // top: 100px;
      // left: 50%;
      // transform: translateX(-50%);

      border: 2px black dotted;
      border-radius: 20px 20px 0px 0px;
      padding: 10px 20px;

      font-size: 20px;
    }

    .app-list {
      width: 100%;

      margin-bottom: 10px;
      padding-bottom: 10px;
      border-bottom: 2px black solid;
      // background-color: red;

      cursor: pointer;

      .app-icon {
        width: 20%;
        // float: left;
        display: inline-block;

        border-radius: 20px;

        box-shadow: 0px 0px 20px 0px rgba(0, 0, 0, 0.5);
      }

      .app-name {
        width: 100%;
        // height: 100%;

        float: right;
        position: relative;
        top: 50%;
        transform: translateY(-50%);
        vertical-align: middle;

        text-align: right;
      }
    }
    .app-list:last-child {
      margin-bottom: 0px;
      padding-bottom: 0px;
      border-bottom: none;
    }

    .query-list {
      width: 100%;
      height: 40vh;

      position: absolute;
      z-index: 999;

      border: 2px black solid;
      border-radius: 0px 0px 20px 20px;
      padding: 10px;

      list-style: none;
      overflow-x: hidden;
      overflow-y: scroll;

      font-size: 20px;
      
      background-color: white;
    }

    .selected-list {
      width: calc(100% - 20px);
      height: 70vh;

      position: absolute;
      left: 10px;

      border: 2px black solid;
      border-radius: 20px;
      padding: 10px;

      list-style: none;
      overflow-x: hidden;
      overflow-y: scroll;

      font-size: 20px;
    }
  }
  
}
</style>