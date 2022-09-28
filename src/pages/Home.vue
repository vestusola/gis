<template>
  <div>
    <div id="map"></div>
  </div>
</template>
<script>
import { Loader } from "@googlemaps/js-api-loader";
import { API_URL } from "../plugins/api";
import { mapSettings } from "../utils/mapSettings";
// import axios from 'axios';
export default {
  name: "home",
  data() {
    return {
      apiKey: "AIzaSyAq2t3VmHFIYc_XM0P-hGii928juQocCVc",
      // libraries: ["drawing", "geometry", "places"],
      libraries: [],
      //mapCenter: { lat: 8.14968146497296, lng: 4.713906499464024 },
      mapCenter: { lat: 8.133669901120484, lng: 4.712130057292575 },
      type: 'department'
    };
  },
  methods: {
    fetchPlaces() {
      this.axios
        .get(`${API_URL}/api/places`, {
          headers: {
            Accept: "*",
            "Access-Control-Allow-Origin": "*",
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          if (response.status == 200) {
            const googleMapApi = new Loader({
              apiKey: this.apiKey,
              libraries: this.libraries,
            });

            googleMapApi.load().then((google) => {
              let map = new google.maps.Map(document.querySelector("#map"), {
                center: this.mapCenter,
                ...mapSettings,
              });

              response.data.data.forEach((item) => {
                let marker = new google.maps.Marker({
                  position: { lat: item.latitude, lng: item.longitude },
                  map: map,
                  title: item.name,
                  visible: true
                });

                var infoWindow = new google.maps.InfoWindow();
                infoWindow.setContent(item.name);
                infoWindow.open(map, marker);

                marker.setMap(map);
                marker.addListener('click', function () {
                  infoWindow.open(map);
                })
              });
              if (this.type == 'department') {
                mapSettings.zoom = 17;
              }
              map.setZoom(mapSettings.zoom);
            });
            console.log(response.data.data);
          } else {
            console.log(response.data.message);
          }
        })
        .catch((err) => console.log(err.message));
    },
  },
  mounted() {
    this.fetchPlaces();
  },
};
</script>
<style>
#map {
  background-color: #ccc;
  height: 100vh !important;
  width: 100%;
  overflow: hidden;
}
</style>
