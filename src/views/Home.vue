<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div
      class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
    >
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">Ip Address Tracker</h1>
        <div class="flex">
          <input
            v-model="queryIp"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            type="text"
            placeholder="Search for any IP Address or leave blank to get your ip info"
          />
          <i
            @click="getIpInfo"
            class="flex items-center cursor-pointer px-4 rounded-tr-md rounded-br-md bg-black text-white fas fa-chevron-right"
          ></i>
        </div>
      </div>
      <!-- Ip Info -->
      <IpInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>

    <!-- Map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
import IpInfo from "../components/IpInfo.vue";
import leaflet from "leaflet";
import { onMounted } from "@vue/runtime-core";
import { ref } from "vue";

export default {
  name: "Home",
  components: { IpInfo },
  setup() {
    let mymap;
    const queryIp = ref("");
    const ipInfo = ref(null);

    onMounted(() => {
      mymap = leaflet.map("mapid").setView([51.505, -0.09], 9);

      leaflet
        .tileLayer(
          "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiZGFubnk2NjExIiwiYSI6ImNrcjJucjJxdDBqM24yb3B0ZDVpNW9sODUifQ.u8n04qbFEBeHZo41NfxF6g",
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken:
              "pk.eyJ1IjoiZGFubnk2NjExIiwiYSI6ImNrcjJucjJxdDBqM24yb3B0ZDVpNW9sODUifQ.u8n04qbFEBeHZo41NfxF6g",
          }
        )
        .addTo(mymap);
    });

    const getIpInfo = async () => {
      try {
        const data = await fetch(
          `https://geo.ipify.org/api/v1?apiKey=at_xFLpKgtlXnXfY0c9Sgu9ZUulop5Ti&ipAddress=${queryIp.value}`
        );
        if (!data.ok) {
          throw Error(
            "ERROR: Could not fetch the IP data, please try again later."
          );
        }
        const result = await data.json();
        ipInfo.value = {
          address: result.ip,
          region: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([result.location.lat, result.location.lng]).addTo(mymap);
        mymap.setView([result.location.lat, result.location.lng], 13);
      } catch (err) {
        alert(err.message);
      }
    };

    return { queryIp, ipInfo, getIpInfo };
  },
};
</script>
