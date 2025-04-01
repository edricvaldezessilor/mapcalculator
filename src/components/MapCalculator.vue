<template>
  <div class="flex flex-col items-center gap-6 p-6 bg-gray-100 min-h-screen">
    <!-- Upload CSV Section -->
    <div class="p-6 w-full max-w-3xl bg-white border rounded-lg shadow-lg transition-transform transform hover:scale-105">
      <h3 class="text-xl font-bold text-center text-gray-700 mb-4">Upload CSV</h3>
      <input
        type="file"
        @change="handleFileUpload"
        accept=".csv"
        class="mt-2 border border-gray-300 p-2 rounded w-full focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        v-if="processedCsv"
        @click="downloadCsv"
        class="bg-green-500 text-white p-3 rounded w-full mt-4 hover:bg-green-600 transition-colors"
      >
        Download Processed CSV
      </button>
    </div>

    <!-- Divider -->
    <div class="w-full max-w-3xl border-t-4 border-blue-500 my-6"></div>

    <!-- Direction Verifier Section -->
    <h3 class="text-2xl font-semibold text-gray-700">Direction Verifier</h3>
    <div class="p-6 w-full max-w-3xl bg-white border rounded-lg shadow-lg transition-transform transform hover:scale-105">
      <div class="grid grid-cols-2 gap-6">
        <div class="flex flex-col gap-4">
          <h3 class="text-lg font-semibold text-gray-600">Origin</h3>
          <input
            type="number"
            v-model.number="lat1"
            placeholder="Latitude 1"
            class="border border-gray-300 p-3 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <input
            type="number"
            v-model.number="lon1"
            placeholder="Longitude 1"
            class="border border-gray-300 p-3 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
        <div class="flex flex-col gap-4">
          <h3 class="text-lg font-semibold text-gray-600">Destination</h3>
          <input
            type="number"
            v-model.number="lat2"
            placeholder="Latitude 2"
            class="border border-gray-300 p-3 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <input
            type="number"
            v-model.number="lon2"
            placeholder="Longitude 2"
            class="border border-gray-300 p-3 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
      </div>
      <button
        @click="calculateDistance"
        class="bg-blue-500 text-white p-3 rounded w-full mt-6 hover:bg-blue-600 transition-colors"
      >
        Calculate Distance
      </button>
      <p
        v-if="distance !== null"
        class="text-center mt-4 text-lg font-medium text-gray-700 animate-fade-in"
      >
        Distance: <span class="font-bold text-blue-500">{{ distance }} km</span> | Estimated Time: <span class="font-bold text-green-500">{{ estimatedTime }} mins</span>
      </p>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import 'leaflet-routing-machine';
import Papa from 'papaparse';

export default {
  data() {
    return {
      lat1: '',
      lon1: '',
      lat2: '',
      lon2: '',
      distance: null,
      estimatedTime: null,
      processedCsv: null,
    };
  },
  methods: {
    toRad(angle) {
      return (Math.PI / 180) * angle;
    },
    haversineDistance(lat1, lon1, lat2, lon2) {
      const R = 6371;
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
        this.estimatedTime = Math.round((this.distance / 50) * 60); // Convert hours to minutes
      }
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        Papa.parse(file, {
          header: true,
          skipEmptyLines: true,
          complete: (result) => {
            const updatedData = result.data.map((row) => {
              const distance = this.haversineDistance(
                parseFloat(row.LATITUDEA),
                parseFloat(row.LONGITUDEA),
                parseFloat(row.LATITUDEB),
                parseFloat(row.LONGITUDEB)
              ).toFixed(2);
              const estimatedTime = Math.round((distance / 50) * 60); // Convert hours to minutes
              return {
                ...row,
                DISTANCE: `${distance} km`,
                TRAVEL_TIME: `${estimatedTime} mins`,
              };
            });
            this.processedCsv = Papa.unparse(updatedData);
          },
        });
      }
    },
    downloadCsv() {
      const blob = new Blob([this.processedCsv], { type: 'text/csv' });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'processed_carmel.csv';
      a.click();
      window.URL.revokeObjectURL(url);
    },
  },
};
</script>

<style>
@keyframes fade-in {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.animate-fade-in {
  animation: fade-in 1s ease-in-out;
}

body {
  font-family: 'Arial', sans-serif;
  background-color: #f9fafb;
}

input:focus {
  box-shadow: 0 0 5px rgba(59, 130, 246, 0.5);
}

.border-t-4 {
  border-top-width: 4px;
}
</style>
