<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇所在區域 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="cityName" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="cityName" class="form-control" v-model="select.city">
              <option value="">請選擇</option>
              <option
                :value="city.CityName"
                v-for="city in cityName"
                :key="city.CityName"
              >
                {{city.CityName}}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="area" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="area" class="form-control" v-model="select.area">
              <option value="">請選擇</option>
              <option
                :value="area.AreaName"
                v-for="area in filterArea"
                :key="area.AreaName"
              >
                {{area.AreaName}}
              </option>
            </select>
          </div>
        </div>
      </div>

      <!-- 顯示藥局位置 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import cityName from './assets/CityCountyData.json';
import axios from 'axios';
import L from 'leaflet';

let openStreetMap = {};

export default {
  name: 'App',
  data () {
    return {
      data: [],
      cityName,
      select: {
        city: '臺北市',
        area: '中正區',
      }
    }
  },
  components: {},
  computed: {
    filterArea () {
      return this.cityName.find((city) => city.CityName === this.select.city).AreaList
    },
    pharmacies () {
      return this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.county === this.select.city
        }
        return pharmacy.properties.town === this.select.area
      })
    }
  },
  watch: {
    pharmacies() {
      this.updateMap()
    }
  },
  mounted() {
    this.initCityArea()
    this.setStreetMap()
  },
  methods: {
    initCityArea () {
      const api = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json'
      axios.get(api).then(res => {
        this.data = res.data.features
      })
    },
    setStreetMap () {
      openStreetMap = L.map('map', {
        center: [25.042474, 121.513729],
        zoom: 18,
      })

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 20,
      }).addTo(openStreetMap);
    },
    updateMap () {
      // 1. 清除上個查詢疊加
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          openStreetMap.removeLayer(layer)
        }
      })
      // 2. 疊加查詢區域
      this.pharmacies.forEach((pharmacy) => {
        L.marker([
            pharmacy.geometry.coordinates[1],
            pharmacy.geometry.coordinates[0],
          ])
          .addTo(openStreetMap)
          .bindPopup(
            `<p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
              <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
              地址: ${pharmacy.properties.address}<br>
              電話: ${pharmacy.properties.phone}<br>
              <small>最後更新時間: ${pharmacy.properties.updated}</small>`
          )
      })
    }
  }
}
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  position: relative;
  height: 100vh;
}
</style>
