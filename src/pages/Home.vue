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
      departmentCoords: { lat: 8.14968146497296, lng: 4.713906499464024 },
      fillingStationCoords: { lat: 8.133669901120484, lng: 4.712130057292575 },
      type: process.env.VUE_APP_MAP_TYPE
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
                center: this.type == 'filling-station' ? this.fillingStationCoords : this.departmentCoords,
                ...mapSettings,
              });

              response.data.data.forEach((item) => {
                let marker = new google.maps.Marker({
                  position: { lat: item.latitude, lng: item.longitude },
                  map: map,
                  title: item.name,
                  visible: true
                });

                
                marker.setMap(map);
                marker.addListener('click', function () {
                  var infoWindow = new google.maps.InfoWindow();
                  infoWindow.setContent(`<h5><b>${item.name}</b></h5><p><b>Address:</b> ${item.address}</p>`)
                  infoWindow.open(map, marker);
                })
              });
              if (this.type == 'department') {
                mapSettings.zoom = 17;
              }
              map.setZoom(mapSettings.zoom);
            });
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
