<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇地區 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="city" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="city" class="form-control" v-model="selected.city">
              <!-- 製作下拉選單 -->
              <option v-for="city in cityMenu" :key="city" :value="city">{{ city }}</option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="dist" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="dist" class="form-control" v-model="selected.area">
              <!-- 製作下拉選單 -->
              <option v-for="area in areaMenu" :key="area" :value="area">{{ area }}</option>
            </select>
          </div>
        </div>
      </div>

      <!-- 顯示地圖和 UBike 站點 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import Taipei from '@/assets/area.json';
import L from 'leaflet';

export default {
  name: 'Ubike-map',
  data() {
    return {
      cityMenu: [],
      areaMenu: [],
      ubikes: [],
      selected: { city: '臺北市', area: '萬華區' },
      osmMap: {},
    };
  },
  computed: {
    selectedArea() {
      const vm = this;
      return vm.ubikes.filter((item) => item.sarea === vm.selected.area);
    },
  },
  watch: {
    selectedArea() {
      const vm = this;
      vm.updateMarkers();
    },
  },
  methods: {
    updateMarkers() {
      const vm = this;

      // 移除標記，使用 instanceof 判斷是否是marker物件，如果是就移除
      vm.osmMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          vm.osmMap.removeLayer(layer);
        }
      });

      // 新增標記、popup
      vm.selectedArea.forEach((area) => {
        L.marker([area.lat, area.lng]).addTo(vm.osmMap).bindPopup(`<p><strong style="font-size: 20px;">${area.sna}</strong></p>
 <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${area.sbi} 台</strong><br>
 可停空位剩餘: ${area.bemp}<br>
 <small>最後更新時間: ${area.mday}</small>`);
      });

      // 移動中心點
      vm.osmMap.panTo([vm.selectedArea[0].lat, vm.selectedArea[0].lng]);
    },
  },
  created() {
    const vm = this;
    vm.cityMenu = [Taipei[0].name];
    vm.areaMenu = Taipei[0].districts.map((area) => area.name);
    const api = 'https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json';
    vm.$http.get(api).then((response) => {
      vm.ubikes = response.data.retVal;
      // 使用 Object.keys獲取{{},{},{},{},{}}的方式
      vm.ubikes = Object.keys(vm.ubikes).map((key) => vm.ubikes[key]);
    });
  },
  mounted() {
    const vm = this;
    // 加入leaflet步驟：https://leafletjs.com/examples/quick-start/
    // OSM地圖參考：https://leafletjs.com/
    vm.osmMap = L.map('map').setView([25.041956, 121.508791], 16);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'your.mapbox.access.token',
    }).addTo(vm.osmMap);
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  background-color: rgb(235, 235, 235);
  height: 100vh;
}
</style>
