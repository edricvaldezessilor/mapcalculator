<template>
  <div class="flex flex-col items-center gap-4 p-4">
    <div class="p-4 w-full max-w-2xl border rounded shadow">
      <div class="grid grid-cols-2 gap-4">
        <div class="flex flex-col gap-2">
          <h3 class="text-lg font-semibold">Point 1</h3>
          <input
            type="number"
            v-model.number="lat1"
            placeholder="Latitude 1"
            class="border p-2 rounded"
          />
          <input
            type="number"
            v-model.number="lon1"
            placeholder="Longitude 1"
            class="border p-2 rounded"
          />
        </div>
        <div class="flex flex-col gap-2">
          <h3 class="text-lg font-semibold">Point 2</h3>
          <input
            type="number"
            v-model.number="lat2"
            placeholder="Latitude 2"
            class="border p-2 rounded"
          />
          <input
            type="number"
            v-model.number="lon2"
            placeholder="Longitude 2"
            class="border p-2 rounded"
          />
        </div>
      </div>
      <button
        @click="calculateDistance"
        class="bg-blue-500 text-white p-2 rounded w-full mt-4"
      >
        Calculate Distance
      </button>
      <p v-if="distance !== null" class="text-center mt-2">
        Distance: {{ distance }} km
      </p>
    </div>
    <!-- <div id="map" class="w-full h-[600px] mt-4"></div> -->
    <!-- <div
      v-if="directions"
      class="w-full max-w-2xl p-4 border rounded shadow mt-4"
    >
      <h3 class="text-lg font-semibold">Directions</h3>
      <ul>
        <li v-for="(step, index) in directions" :key="index">{{ step }}</li>
      </ul>
    </div> -->
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import 'leaflet-routing-machine';

export default {
  data() {
    return {
      lat1: '',
      lon1: '',
      lat2: '',
      lon2: '',
      distance: null,
      map: null,
      routeControl: null,
      directions: null,
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    toRad(angle) {
      return (Math.PI / 180) * angle;
    },
    haversineDistance(lat1, lon1, lat2, lon2) {
      const R = 6371; // Earth radius in km
      const dLat = this.toRad(lat2 - lat1);
      const dLon = this.toRad(lon2 - lon1);
      const a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(this.toRad(lat1)) *
          Math.cos(this.toRad(lat2)) *
          Math.sin(dLon / 2) *
          Math.sin(dLon / 2);
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      return R * c;
    },
    calculateDistance() {
      if (this.lat1 && this.lon1 && this.lat2 && this.lon2) {
        this.distance = this.haversineDistance(
          parseFloat(this.lat1),
          parseFloat(this.lon1),
          parseFloat(this.lat2),
          parseFloat(this.lon2)
        ).toFixed(2);
        this.updateMap();
      }
    },
    initMap() {
      this.map = L.map('map').setView([0, 0], 2);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors',
      }).addTo(this.map);
    },
    updateMap() {
      if (!this.map) return;
      this.map.eachLayer((layer) => {
        if (!!layer.toGeoJSON) {
          this.map.removeLayer(layer);
        }
      });
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors',
      }).addTo(this.map);
      L.marker([this.lat1, this.lon1]).addTo(this.map).bindPopup('Point 1');
      L.marker([this.lat2, this.lon2]).addTo(this.map).bindPopup('Point 2');
      this.map.fitBounds([
        [this.lat1, this.lon1],
        [this.lat2, this.lon2],
      ]);

      if (this.routeControl) {
        this.map.removeControl(this.routeControl);
      }
      this.routeControl = L.Routing.control({
        waypoints: [
          L.latLng(this.lat1, this.lon1),
          L.latLng(this.lat2, this.lon2),
        ],
        routeWhileDragging: true,
        createMarker: () => null, // Hides the text directions inside the map
      }).addTo(this.map);

      this.routeControl.on('routesfound', (e) => {
        this.directions = e.routes[0].instructions.map((instr) => instr.text);
      });
    },
  },
};
</script>

<style>
#map {
  width: 100%;
  height: 600px;
}
input {
  width: 100%;
}
</style>
