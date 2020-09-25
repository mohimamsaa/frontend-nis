<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div class="row">
      <div class="col-md-9">
        <!-- The map goes here -->
          <div id="map"></div>
      </div>
      <div class="col-md-3 "  >
        <!-- The layer checkboxes go here -->
        <div
          class="form-check d-flex justify-content-between"
          v-for="layer in layers"
          :key="layer.id"
        >
          <label class="form-check-label">
            <input
              class="form-check-input"
              type="checkbox"
              v-model="layer.active"
              @change="layerChanged(layer.id, layer.active)"
            />
            {{ layer.name }}
          </label>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import L from 'leaflet';
import axios from 'axios';

export default {
  name: 'App',
  components: {
    // HelloWorld
  },
  data: function() {
    return {
      map: null,
      tileLayer: null,
      layers: [
        {
          id: 0,
          name: 'Point',
          active: false,
          features: [],
        },
        {
          id: 1,
          name: 'Line',
          active: false,
          features: []
        },
        {
          id: 2,
          name: 'Polygon',
          active: false,
          features: []
        },
      ],
    }
  },
  methods: {
    initMap: function () {
      console.log("init")
      this.map = L.map('map').setView([-2, 118], 5);
      this.tileLayer = L.tileLayer(
        'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png',
        {
          maxZoom: 18,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>',
        }
      );
      this.tileLayer.addTo(this.map);
    },
    initData: async function () {
      for (var i = 0;i < 3; i++) {
        const data =  await this.fetchData(this.layers[i].name.toLowerCase())
        this.layers[i].features = data.features
      }

      console.log(this.layers)
      
    },
    initLayer: function () {
      this.layers.forEach((layer) => {
        const markerFeatures = layer.features.filter(feature => feature.geometry.type === 'Point');
        const polygonFeatures = layer.features.filter(feature => feature.geometry.type === 'Polygon');
        const lineFeatures = layer.features.filter(feature => feature.geometry.type === 'LineString')

        markerFeatures.forEach((feature) => {
          const coordinates = []
          coordinates.push(feature.geometry.coordinates[1])
          coordinates.push(feature.geometry.coordinates[0])
          feature.leafletObject = L.marker(coordinates)
            .bindPopup(feature.properties.name);
        });

        polygonFeatures.forEach((feature) => {
          const arrCoordinates = []
          feature.geometry.coordinates[0].forEach((coordinates) => {
            const coor = [coordinates[1], coordinates[0]]
            arrCoordinates.push(coor)
          })
          feature.leafletObject = L.polygon(arrCoordinates)
            .bindPopup(feature.properties.name + " dengan luas " + feature.properties.area);
        });

        lineFeatures.forEach((feature) => {
          const arrCoordinates = []
          feature.geometry.coordinates.forEach((coordinates) => {
            const coor = [coordinates[1], coordinates[0]]
            arrCoordinates.push(coor)
          })
          feature.leafletObject = L.polyline(arrCoordinates)
            .bindPopup(feature.properties.name + " dengan panjang " + feature.properties.long);
        })

      });
    },
    layerChanged(layerId, active) {
      const layer = this.layers.find(layer => layer.id === layerId);
      layer.features.forEach((feature) => {
        if (active) {
          feature.leafletObject.addTo(this.map);
        } else {
          feature.leafletObject.removeFrom(this.map);
        }
      });
    },
    async fetchData(type) {
      const res = await axios.get("http://localhost:5000/api/v1/get-features?type=" + type)
      console.log(type, res)
      if (res.status == 200) {
        return res.data.data
      } else {
        console.log(res.error)
      }
    }

  },

  async mounted () {
    this.initMap()
    await this.initData()
    this.initLayer()
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#map { height: 600px; }
</style>
