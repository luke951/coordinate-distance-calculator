<template>
  <div class="container">
    <h1 class="text-center mb-4">
      Calculate distance between two geographic points
    </h1>
    <div class="row align-items-start">
      <div class="col">
        <div id="map" class="map mb-4"></div>
        <p v-if="distanceInMeters">
          <strong
            >Distance: {{ distanceInMeters }} meters ({{
              distanceInKilometers
            }}
            kilometers)</strong
          >
        </p>
      </div>
      <div class="col">
        <div class="p-3 box">
          <h2 class="mb-3">Form</h2>
          <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
          <form @submit="calculateDistance">
            <div class="row align-items-start">
              <div class="col">
                <div class="mb-3">
                  <label for="point1lat" class="form-label"
                    >Point 1 latitude</label
                  >
                  <input
                    id="point1lat"
                    v-model="point1lat"
                    type="text"
                    class="form-control"
                    :class="[
                      'form-control',
                      { 'is-invalid': errorMessage && point1lat === null },
                    ]"
                    placeholder="lat"
                    min="-180"
                    max="180"
                  />
                  <small class="form-text text-muted">e.g. 52.229675</small>
                </div>
              </div>
              <div class="col">
                <div class="mb-3">
                  <label for="point1lng" class="form-label"
                    >Point 1 longitude</label
                  >
                  <input
                    id="point1lng"
                    v-model="point1lng"
                    type="text"
                    :class="[
                      'form-control',
                      { 'is-invalid': errorMessage && point1lng === null },
                    ]"
                    placeholder="lng"
                    min="-180"
                    max="180"
                  />
                  <small class="form-text text-muted">e.g. 21.012230</small>
                </div>
              </div>
            </div>
            <hr />
            <div class="row align-items-start p-2 mb-4">
              <div class="col">
                <div class="mb-3">
                  <label for="point2lat" class="form-label"
                    >Point 2 latitude</label
                  >
                  <input
                    id="point2lat"
                    v-model="point2lat"
                    type="text"
                    :class="[
                      'form-control',
                      { 'is-invalid': errorMessage && point2lat === null },
                    ]"
                    placeholder="lat"
                    min="-180"
                    max="180"
                  />
                  <small class="form-text text-muted">e.g. 52.1</small>
                </div>
              </div>
              <div class="col">
                <div class="mb-3">
                  <label for="point2lng" class="form-label"
                    >Point 2 longitude</label
                  >
                  <input
                    id="point2lng"
                    v-model="point2lng"
                    type="text"
                    :class="[
                      'form-control',
                      { 'is-invalid': errorMessage && point2lng === null },
                    ]"
                    placeholder="lng"
                    min="-180"
                    max="180"
                  />
                  <small class="form-text text-muted">e.g. 21.1</small>
                </div>
              </div>
            </div>

            <button class="btn btn-success me-2" type="submit">
              Calculate distance
            </button>
            <button
              @click.prevent="resetForm"
              class="btn btn-warning"
              type="button"
            >
              Reset
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet/dist/leaflet.css";

export default {
  data() {
    return {
      map: null,
      distanceInMeters: 0,
      distanceInKilometers: 0,
      point1lat: null,
      point1lng: null,
      point2lat: null,
      point2lng: null,
      lines: null,
      marker1: null,
      marker2: null,
      errorMessage: null,
    };
  },
  mounted() {
    this.initializeMap();
  },
  methods: {
    initializeMap() {
      if (!this.map) {
        this.map = L.map("map").setView([52.229675, 21.01223], 13);
      }

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors',
      }).addTo(this.map);
    },
    calculateDistance(event) {
      event.preventDefault();
      const point1lat = parseFloat(this.point1lat);
      const point1lng = parseFloat(this.point1lng);
      const point2lat = parseFloat(this.point2lat);
      const point2lng = parseFloat(this.point2lng);
      if (!this.point1lat || !this.point1lng || !this.point2lat || !this.point2lng) {
        this.errorMessage = "All fields are required";
        return;
      }
      if (
        isNaN(point1lat) ||
        isNaN(point1lng) ||
        isNaN(point2lat) ||
        isNaN(point2lng) ||
        point1lat < -90 ||
        point1lat > 90 ||
        point1lng < -180 ||
        point1lng > 180 ||
        point2lat < -90 ||
        point2lat > 90 ||
        point2lng < -180 ||
        point2lng > 180
      ) {
        this.errorMessage =
          "Invalid input: Latitude must be between -90 and 90, longitude between -180 and 180.";
        return;
      }
      this.errorMessage = null;
      this.resetMap();
      const distance = L.latLng(point1lat, point1lng).distanceTo(
        L.latLng(point2lat, point2lng)
      );
      this.map.setView([point1lat, point1lng], 10);
      this.marker1 = L.marker([point1lat, point1lng]).addTo(this.map);
      this.marker2 = L.marker([point2lat, point2lng]).addTo(this.map);
      const latlngs = [
        [point1lat, point1lng],
        [point2lat, point2lng],
      ];

      this.lines = L.polyline(latlngs, { color: "blue" }).addTo(this.map);
      this.distanceInMeters = distance.toFixed(2);
      this.distanceInKilometers = (distance / 1000).toFixed(2);
    },
    resetForm() {
      this.resetMap();
      this.point1lat = null;
      this.point1lng = null;
      this.point2lat = null;
      this.point2lng = null;
      this.distanceInMeters = null;
      this.distanceInKilometers = null;
      this.errorMessage = null;
    },
    resetMap() {
      if (this.lines) this.map.removeLayer(this.lines);
      if (this.marker1) this.map.removeLayer(this.marker1);
      if (this.marker2) this.map.removeLayer(this.marker2);
    },
  },
};
</script>

<style>
.map {
  height: 400px;
}
.error {
  display: inline-block;
  background-color: #f1352e;
  color: #fff;
  border-radius: 3px;
  padding: 4px 7px;
  margin-top: 8px;
  font-size: 14px;
}
.form-text {
  font-size: 11px;
}

.box {
  border: 1px solid #ddd;
  border-radius: 4px;
}
</style>
