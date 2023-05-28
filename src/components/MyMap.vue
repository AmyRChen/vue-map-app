<template>
  <div>
    <button class="button" @click="getCurrentLocation">Get Current Location</button>
    <div class="current-address" v-if="currentAddress">
      Current Address: {{ currentAddress }}
    </div>
    <div class="search-container">
      Search a location:
      <input
        class="search-input"
        type="text"
        v-model="searchQuery"
        @keydown.enter="searchLocation"
        placeholder="location..."
        ref="searchInput"
      />
      <button class="button" @click="searchLocation">Search</button>
    </div>
    <div class="time-zone">
      The latest searched location: <span class="highlight">{{ latestSearchedLocation }}</span> (<span class="highlight">{{ timeZone }}</span>)
      | Current Time: <span class="highlight">{{ currentTime }}</span>
    </div>
    <div class="map-container">
      <div ref="map" class="map"></div>
    </div>
    <!--listen for the delete-selected event emitted by LocationTable.vue -->
    <location-table
      :locations="searchedLocations"
      :page-size="pageSize"
      @delete-selected="deleteSelectedHistory"
    ></location-table>
  </div>
</template>

<script>
import LocationTable from '@/components/LocationTable.vue';
/* eslint-disable no-undef */ // Add this line to disable ESLint error for 'google'

export default {
  components: {
    LocationTable,
  },
  data() {
    return {
      searchQuery: '',
      searchedLocations: [],
      pageSize: 10,
      markerMap: new Map(),
      latestSearchedLocation: '',
      timeZone: 'UTC', // Initialize with a default time zone
      currentTime: '',
      intervalId: null, // variable to store the interval ID
      currentAddress:'',
    };
  },
  mounted() {
    this.initializeMap();
  },
  computed: {
    hasSearchedLocations() {
      return this.searchedLocations.length > 0;
    },
  },
  methods: {
    initializeMap() {
      // Initialize the map with a default location
      const defaultLocation = { lat: 43.651070, lng: -79.347015 }; // Toronto coordinates
      const mapOptions = {
        center: defaultLocation,
        zoom: 12,
      };
      this.map = new google.maps.Map(this.$refs.map, mapOptions);
    },
    getCurrentLocation() {
      // Acquire the user's current location
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          position => {
            const { latitude, longitude } = position.coords;
            const currentLocation = new google.maps.LatLng(latitude, longitude);
            this.map.setCenter(currentLocation);
            const marker = new google.maps.Marker({ position: currentLocation, map: this.map });
            this.markerMap.set('currentLocation', marker);

            // Get the current address using Geocoding
            const geocoder = new google.maps.Geocoder();
            geocoder.geocode({ location: currentLocation }, (results, status) => {
              if (status === 'OK' && results[0]) {
                this.currentAddress = results[0].formatted_address;
              } else {
                console.log('Geocode was not successful for the following reason:', status);
              }
            });
          },
          error => {
            console.log('Error retrieving location:', error);
          }
        );
      } else {
        console.log('Geolocation is not supported by this browser.');
      }
    },
    searchLocation() {
      const geocoder = new google.maps.Geocoder();
      const searchAddress = this.searchQuery;
      geocoder.geocode(
        { address: searchAddress },
        (results, status) => {
          if (status === 'OK' && results[0]) {
            const { location } = results[0].geometry;
            this.map.setCenter(location);
            const marker = new google.maps.Marker({ position: location, map: this.map });
            this.markerMap.set(searchAddress, marker);
            this.searchedLocations.push(searchAddress);
            this.latestSearchedLocation = searchAddress; // Update the latest searched location

            // Get the time zone and current time for the latest searched location
            this.getTimeZoneAndCurrentTime(location.lat(), location.lng());
          } else {
            console.log('Geocode was not successful for the following reason:', status);
          }
          this.searchQuery = '';
          this.$refs.searchInput.value = '';
        }
      );
    },
    getTimeZoneAndCurrentTime(latitude, longitude) {
      const timestamp = Math.floor(Date.now() / 1000);
      const apiEndpoint = `https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${timestamp}&key=AIzaSyB94SJ-dNFOIwtba1duGA2F9buYg4L5-Wc`;

      fetch(apiEndpoint)
        .then(response => response.json())
        .then(data => {
          const { timeZoneId } = data;
          this.timeZone = timeZoneId || 'UTC'; // Use default value if timeZoneId is empty;

          const currentTime = new Date();
          const formattedCurrentTime = currentTime.toLocaleString('en-US', {
            timeZone: timeZoneId,
          });
          this.currentTime = formattedCurrentTime;
        })
        .catch(error => {
          console.log('Error retrieving time zone:', error);
          this.timeZone = 'UTC'; // Handle error by setting a default time zone
        });
        
        // Call the updateCurrentTime function immediately to set the initial time
        this.updateCurrentTime();

        // Update the current time every second using setInterval
        this.intervalId = setInterval(() => {
          this.updateCurrentTime();
        }, 1000);   
    },
    updateCurrentTime() {
      const currentTime = new Date();
      const formattedCurrentTime = currentTime.toLocaleString('en-US', {
        timeZone: this.timeZone,
      });
      this.currentTime = formattedCurrentTime;
    },
    deleteSelectedHistory(selectedLocations) {
      selectedLocations.forEach(location => {
        const markerKey = location === 'currentLocation' ? location : location.toLowerCase();
        const marker = this.markerMap.get(markerKey);
        if (marker) {
          marker.setVisible(false);
          this.markerMap.delete(markerKey);
        }

        const index = this.searchedLocations.indexOf(location);
        if (index !== -1) {
          this.searchedLocations.splice(index, 1);
        }
      });
      // Clear the interval when all locations are deleted
      if (this.searchedLocations.length === 0) {
        clearInterval(this.intervalId);
      }
    },
  },
};
</script>


<style >
.map-container {
  width: 80%;
  height: 500px;
  margin: 0 auto;
}
.map {
  width: 100%;
  height: 100%;
}

.button{
  margin: 20px 10px;
  display: inline-block;
  outline: none;
  cursor: pointer;
  font-size: 14px;
  padding: 0 12px;
  line-height: 20px;
  height: 30px;
  max-height: 30px;
  background: #f0fbfa;
  font-weight: 700;
  border: 2px solid #8ad7de;
  border-radius: 15px;
  color: #1fa0aa;
  transition-timing-function: ease-in-out;
  transition-property: box-shadow;
  transition-duration: 150ms;
}

.button:hover {
  box-shadow: 0 2px 2px rgb(39 44 52 / 12%);
}

.search-input{
  margin-left: 5px;
}

.time-zone{
  margin: 25px 0;
  color: #1fa0aa;
}

.search-container{
  margin: 25px 0;
  color: #1fa0aa;
}

.highlight{
  color:#ea9c3d;
}
</style>