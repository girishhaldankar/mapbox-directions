<template>
  <div>
    <input
      type="text"
      v-model="destinationInput"
      placeholder="Enter destination"
      @keydown.enter="searchDestination"
    />
    <button @click="startTrackingLocation">Track My Location</button>
    <div id="map-container" ref="mapContainer"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl';
import MapboxDirections from '@mapbox/mapbox-gl-directions/dist/mapbox-gl-directions';
import '@mapbox/mapbox-gl-directions/dist/mapbox-gl-directions.css';

export default {
  name: 'MapComponent',
  data() {
    return {
      map: null,
      directions: null,
      destinationInput: '',
      userLocation: null,
      destinationCoordinates: null,
      locationWatchId: null,
    };
  },
  mounted() {
    mapboxgl.accessToken = 'pk.eyJ1IjoibWVycmlsbGdvbnNhbHZlcyIsImEiOiJjajN6ampkbzcwM3VxMzJyemR5Y2dqdTZuIn0.EGosEkXFJM5K_82Vy0XuCA';

    this.map = new mapboxgl.Map({
      container: this.$refs.mapContainer,
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [-74.006, 40.7128],
      zoom: 12,
    });

    this.directions = new MapboxDirections({
      accessToken: mapboxgl.accessToken,
      unit: 'metric',
      profile: 'mapbox/driving',
      controls: { instructions: true },
    });

    // Add Directions control to the map in the top-left
    this.map.addControl(this.directions, 'top-left');

    this.map.on('load', () => {
      console.log('Map has loaded successfully!');
    });
  },
  methods: {
    startTrackingLocation() {
      if (!navigator.geolocation) {
        alert('Geolocation is not supported by your browser.');
        return;
      }
      if (this.locationWatchId) {
        navigator.geolocation.clearWatch(this.locationWatchId);
      }

      this.locationWatchId = navigator.geolocation.watchPosition(
        (position) => {
          this.userLocation = [
            position.coords.longitude,
            position.coords.latitude,
          ];
          this.directions.setOrigin(this.userLocation);
          this.map.setCenter(this.userLocation);
        },
        (error) => {
          console.error('Error getting location:', error);
          alert('Could not retrieve your location.');
        },
        {
          enableHighAccuracy: true,
          maximumAge: 1000,
          timeout: 5000,
        }
      );
    },
    async searchDestination() {
      if (!this.destinationInput) {
        alert('Please enter a destination.');
        return;
      }

      const url = `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(
        this.destinationInput
      )}.json?access_token=${mapboxgl.accessToken}`;

      try {
        const response = await fetch(url);
        const data = await response.json();
        if (data.features && data.features.length > 0) {
          this.destinationCoordinates = data.features[0].geometry.coordinates;
          this.directions.setDestination(this.destinationCoordinates);
          this.map.setCenter(this.destinationCoordinates);
        } else {
          alert('Location not found.');
        }
      } catch (error) {
        console.error('Error fetching destination:', error);
      }
    },
  },
  beforeUnmount() {
    if (this.locationWatchId) {
      navigator.geolocation.clearWatch(this.locationWatchId);
    }
  },
};
</script>

<style>
#map-container {
  width: 100%;
  height: 80vh;
  margin-top: 10px; position: relative;
}
input {
  margin-bottom: 10px;
  padding: 5px;
}
button {
  margin-left: 10px;
  padding: 5px 10px;
}

/* Override styles to keep Directions control in the top-left */
.mapboxgl-ctrl-top-left {
  position: absolute; top: 0;
  display: flex !important;
  flex-direction: column;
  align-items: flex-start;
}

.mapboxgl-ctrl-directions {
  position: relative !important;
  z-index: 1;
  margin: 10px;
}
</style>
