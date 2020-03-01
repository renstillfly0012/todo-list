<template>
  <div>
    <q-toolbar class="bg-primary text-white">
      <q-btn flat round dense icon="menu" class="q-mr-sm" />
      <q-avatar>
        <img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg" />
      </q-avatar>

      <q-toolbar-title>GeoMap</q-toolbar-title>

      <q-btn flat round dense icon="map" />
    </q-toolbar>
    <div class="bg-grey" style="width: 100vw; height: calc(100vh - 50px);">
      <l-map :zoom="zoom" :center="center" :options="mapOptions">
        <l-tile-layer
          url='https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
          attribution='&copy;
        <a href=http://osm.org/copyright"
        >OpenStreetMap</a> contributors'
        />
        <l-marker :lat-lng="center">
            <l-icon :icon-size="[32,37]"  :icon-url="icon"/>
        </l-marker>

      </l-map>
    </div>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker, LIcon } from "vue2-leaflet";
import { latLng } from "leaflet";
import "leaflet/dist/leaflet.css";

export default {
  beforeDestroy() {
    navigator.geolocation.clearWatch(this.positionWatcher);
  },
  mounted() {
    console.log(" hi hello");
    const options = {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0
    }
    navigator.geolocation.getCurrentPosition(
      this.setLocation,
      this.locationError,
      options
    )
    this.positionWatcher = navigator.geolocation.watchPosition(
      this.setLocation,
      this.locationError,
      options
    )
  },
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LIcon,
  },

  data() {
    return {
        icon: require('assets/mob.jpg'),
      positionWatcher: null,
      zoom: 13,
      center: latLng(47.41322, -1.219482),
      mapOptions: {
        zoomSnap: 0.5
      }
    }
  },
  methods: {
    setLocation(pos) {
         console.log(" hello");
      const { latitude, longitude } = pos.coords;
      this.center = latLng(latitude, longitude);
    },
    locationError() {}
  }
};
</script>