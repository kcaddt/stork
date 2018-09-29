<template>
  <div id="app">
    <div id="search">
      <img src="./assets/stork.png" />
      <input v-model="query" placeholder="Search a city">
    </div>

    <transition name="fade">
      <div id="travel" v-if="totalDistance >0">
          <b>{{totalDistance.toLocaleString("fr")}}</b> km parcourus
      </div>
    </transition>
    <ul id='journey' v-if="cities.length > 0">
      <template v-for="city in cities">
        <li><i class="fas fa-caret-right"></i> {{city}}</li>
      </template>
    </ul>
    <ais-index :query="query" app-id="W7ILHFZ3PU" api-key="0bc8205dcd63b9929bd34ed140c76488" index-name="cities">
      <ais-results v-if="query.length >0" id="search-results">
        <ul slot-scope="{ result }">
          <li v-on:click="search(result)"><i class="fas fa-caret-right"></i> {{result.name}} <span class="city">{{result.country}}</span></li>
        </ul>
      </ais-results>
    </ais-index>

    <div id="search-info" v-if="(query.length < 1) && (!!selected)">
      <h2><i class="fas fa-globe-africa"></i> {{selected.name}}</h2>
      <h3>{{selected.country}}</h3>
      <h4>Size :
        <span v-for="i in Math.round(Math.log10(selected.population/1000)) + 1">
         <i class="fas fa-building"></i>
       </span>
       </h4>
      <h4> Population : {{selected.population.toLocaleString('fr')}} people</h4>
    </div>

    <div id='map'></div>

  </div>
</template>

<script>
export default {
  name: "app",
  data: function() {
    return {
      geo: [2.34, 48.86],
      query: "",
      selected: null,
      totalDistance: 0,
      cities: []
    };
  },
  mounted: function() {
    mapboxgl.accessToken =
      "pk.eyJ1IjoibHZyYyIsImEiOiJjam1sczJ5aXIwYmZ2M3dyMmxxcnFwd3psIn0.YvVKReBgmYcIY-qyf1hw7Q";
    this.map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/mapbox/streets-v9",
      center: this.geo,
      zoom: 4
    });
  },
  methods: {
    search: function(result) {
      this.query = "";
      const newCordinates = [result.lng, result.lat];

      this.cities.push(result.name + " (" + result.country + ")");

      if (!!this.selected) {
        const from = turf.point([this.selected.lng, this.selected.lat]);
        const to = turf.point(newCordinates);
        this.totalDistance += Math.round(
          turf.distance(from, to, { units: "kilometers" })
        );
      }
      this.selected = result;
      this.map.flyTo({ center: newCordinates, zoom: 12 });
    }
  }
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

#search {
  width: 350px;
  height: 40px;
  padding: 20px 10px 10px 20px;
  background-color: #fff;
  top: 40px;
  left: 40px;
  position: absolute;
  z-index: 10;
}

#travel {
  width: 200px;
  height: 20px;
  padding: 15px 20px 15px 20px;
  position: absolute;
  z-index: 10;
  background-color: #fff;
  top: 40px;
  right: 40px;
  text-align: right;
}

#journey {
  width: 200px;
  text-transform: uppercase;
  list-style: none;
  font-weight: bold;
  font-size: 13px;
  padding: 15px 20px 15px 20px;
  position: absolute;
  z-index: 10;
  background-color: #fff;
  top: 90px;
  right: 40px;
  opacity: 0.8;
}

#journet > ul > li {
  padding-top: 10px;
}
#search input {
  border: 0px solid #fff;
  font-size: 16px !important;
  margin-top: 5px;
}
#search img {
  float: left;
  height: 40px;
  margin-top: -6px;
  margin-right: 25px;
}
#search-results {
  width: 380px;
  background-color: #fff;
  top: 120px;
  left: 40px;
  position: absolute;
  z-index: 10;
  max-height: 600px;
  overflow: auto;
}
#search-info {
  width: 340px;
  height: 150px;
  background-color: #fff;
  top: 120px;
  left: 40px;
  position: absolute;
  z-index: 10;
  padding: 20px 20px 20px 20px;
}
#search-info .close {
  float: right;
  margin-top: 13px;
  color: #95a5a6;
}

#search-info h2 {
  font-size: 17px;
  text-transform: uppercase;
  margin-bottom: -10px !important;
  color: #409eff;
  font-weight: bolder;
}
#search-info h3 {
  font-size: 15px;
  font-weight: normal;
  margin-left: 30px;
}
#search-info h2 i {
  margin-right: 7px;
  color: #409eff;
}
#search-info h4 {
  font-size: 15px;
  margin-left: 30px;
  margin-bottom: -10px;
}
#search-info h4 i {
  margin-right: 5px;
  font-size: 12px;
  color: #2c3e50;
}
#search-info p {
  margin-left: 30px;
  margin-top: 40px;
}

#search-results ul {
  list-style: none;
  overflow: auto;
  padding: 0;
  margin: 0;
}
#search-results li {
  padding: 20px 0 15px 20px;
  text-transform: uppercase;
  font-size: 14px;
  font-weight: bold;
}
#search-results ul > li > .city {
  font-weight: normal;
  float: right;
  margin-right: 15px;
  font-size: 12px;
}
#search-results ul > li > i {
  color: #bdc3c7;
  margin-right: 5px;
}

#search-results li:hover {
  background-color: #ecf0f1;
}

#map {
  position: absolute;
  z-index: 8;
  top: 0;
  bottom: 0;
  width: 100%;
}

.ais-search-box__submit {
  display: none;
}

.ais-clear {
  margin-left: 5px;
  opacity: 0.5;
}

.ais-clear:hover {
  margin-left: 5px;
  opacity: 1;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
