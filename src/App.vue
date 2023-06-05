<script setup lang="ts">
import { onMounted, ref } from "vue";
import "leaflet/dist/leaflet.css"
import L from 'leaflet'

import { subdivision } from 'iso-3166-2';

const stateAbbreviations = {

}

let ipAddress = ref('')
let region = ref('')
let regionCode = ref('')
let city = ref('')
let timezone = ref('')
let postalCode = ref('')
let isp = ref('')
let lat = ref()
let lng = ref()
let countryCode = ref('')
let map = ref<L.Map | any>({})
const key = import.meta.env.VITE_IPFY_KEY

function getLocation() {
  fetch(`https://geo.ipify.org/api/v2/country,city?apiKey=${key}&ipAddress=${ipAddress.value}`)
    .then(res => {
      return res.json()
    })
    .then(data => {
      region.value = data.location.region
      city.value = data.location.city
      timezone.value = data.location.timezone
      postalCode.value = data.location.postalCode
      isp.value = data.isp
      lat.value = data.location.lat
      lng.value = data.location.lng
      mapMove()
      console.log(data)
      regionCode.value = convertRegionToAbbreviation(data.location.country, data.location.region)
      // map.panTo([data.location.lat, data.location.lng])
    })
    .catch(error => {
      console.log(error)
    })
}

function convertRegionToAbbreviation(country: string, region: string) {
  let subdivisions = subdivision(country, region)
  if (subdivisions?.regionCode) {
    return subdivisions.regionCode
  } else {
    return region
  }
}

onMounted(() => {
  map = L.map('map').setView([51.505, -0.09], 13);
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);
})

function mapMove() {
  map.panTo([lat.value, lng.value])
  var myIcon = L.icon({
    iconUrl: '/images/icon-location.svg'
  })
  L.marker([lat.value, lng.value], { icon: myIcon }).addTo(map);
}

</script>

<template>
  <div class="container">
    <div class="top">
      <h1>IP Address Tracker</h1>
      <div class="search">
        <form @submit.prevent="getLocation">
          <input type="text" placeholder="Search for any IP address or domain" v-model="ipAddress">
          <button :disabled="ipAddress === ''" type="submit" class="submitBtn"><img src="/images/icon-arrow.svg"></button>
        </form>
      </div>
    </div>

    <div class="infoContainer">
      <div class="ipAddress">
        <span class="title">IP Address</span>
        <p v-if="ipAddress && isp" class="info">{{ ipAddress }}</p>
      </div>
      <div class="location">
        <span class="title">Location</span>
        <p v-if="region && city" class="info">{{ city }}, {{ regionCode }}</p>
        <p v-if="postalCode" class="info">{{ postalCode }}</p>
      </div>
      <div class="timezone">
        <span class="title">Timezone</span>
        <p v-if="timezone" class="info">UTC {{ timezone }}</p>
      </div>
      <div class="isp">
        <span class="title">ISP</span>
        <p v-if="isp" class="info">{{ isp }}</p>
      </div>
    </div>

    <div id="map">
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  min-height: 100%;
}

.top {
  background-image: url('/images/pattern-bg-desktop.png');
  background-size: cover;
  height: 30vh;
  /* height: 181px; */
  display: flex;
  flex-direction: column;
}



#map {
  height: 70vh;
}

.search {
  /* margin-left: auto; */
  /* margin-right: auto; */
  margin-top: 1rem;
  display: flex;
  width: 100%;
  justify-content: center;
  align-items: center;
}

.search form {
  display: flex;
  align-items: center;
  justify-content: center;
}

.search button {
  background-color: var(--very-dark-gray);
  color: white;
  border: transparent;
  border-radius: 0px 10px 10px 0px;
  padding: 16.5px;
  cursor: pointer;
}

.search button:hover {
  background-color: var(--dark-gray);
}

.search input {
  border-radius: 10px 0px 0px 10px;
  width: 400px;
  padding: 14px;
  font-size: 18px;
  cursor: pointer;
  border: transparent;
}

.search input:focus,
.search input:focus-visible {
  outline: transparent;
}

.top h1 {
  color: white;
  text-align: center;
  font-weight: 500;
  margin-top: 1rem;
}

.infoContainer {
  position: absolute;
  background-color: white;
  width: 70%;
  margin: auto;
  right: 0;
  left: 0;
  top: 22vh;
  z-index: 999;
  padding: 1rem;
  border-radius: 10px;
  display: flex;
  justify-content: space-around;
}

.infoContainer div {
  width: 100%;
  margin: 1rem;
}

.ipAddress {
  border-right: 1px solid var(--dark-gray);
}

.location {
  border-right: 1px solid var(--dark-gray);
}

.timezone {
  border-right: 1px solid var(--dark-gray);
}

.title {
  color: var(--dark-gray);
  font-size: 12px;
  letter-spacing: 1px;
  text-transform: uppercase;
  font-weight: 800;
}

.info {
  color: var(--very-dark-gray);
  font-size: 23px;
  font-weight: 500;
}

@media (max-width: 500px) {
  .top {
    background-image: url('/images/pattern-bg-mobile.png');
    /* background-image: url('/images/pattern-bg-desktop.png'); */
  }

  .infoContainer {
    flex-direction: column;
    justify-content: center;
    text-align: center;
    width: 92%;
    top: 17vh;
  }

  .ipAddress {
    border-right: 0px solid var(--dark-gray);
  }

  .location {
    border-right: 0px solid var(--dark-gray);
  }

  .timezone {
    border-right: 0px solid var(--dark-gray);
  }

  .infoContainer div {
    margin: 0;
  }

  .search form {
    width: 100%;
    display: flex;
    justify-content: center;
  }

  .search input {
    width: 80%;
  }

}
</style>
