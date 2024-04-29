<template>
  <div class="card">
    <div class="card-header" style="text-align: right">عنوان الأستلام</div>
    <div class="card-body">
      <form @submit.prevent="saveAddress" autocomplete="off">
        <div class="form-label-group mb-3" style="text-align: right">
          <label for="name"> الأسم </label>
          <input
            style="text-align: right"
            type="text"
            id="name"
            v-bind:class="
              errors && errors.name ? 'form-control is-invalid' : 'form-control'
            "
            v-model="user.name"
            placeholder="الأسم"
            required
          />
          <span class="text-danger" v-if="errors && errors.name">
            {{ errors.name[0] }}
          </span>
        </div>
        <!-- <div class="form-label-group mb-3" style="text-align: right">
          <label for="email"> البريد الألكترونى </label>
          <input
            style="text-align: right"
            type="email"
            id="email"
            v-bind:class="
              errors && errors.email
                ? 'form-control is-invalid'
                : 'form-control'
            "
            v-model="user.email"
            placeholder="البريد الألكترونى"
          />
          <span class="text-danger" v-if="errors && errors.email">
            {{ errors.email[0] }}
          </span>
        </div> -->
        <div class="form-label-group mb-3" style="text-align: right">
          <label for="phone"> رقم الهاتف </label>
          <input
            style="text-align: right"
            type="text"
            id="phone"
            v-bind:class="
              errors && errors.phone
                ? 'form-control is-invalid'
                : 'form-control'
            "
            v-model="user.phone"
            placeholder="رقم الهاتف"
            required
          />
          <span class="text-danger" v-if="errors && errors.phone">
            {{ errors.phone[0] }}
          </span>
        </div>
      
        <div class="form-label-group mb-3" style="text-align: right">
          <label for="address">
            عنوان الشحن
          </label>
        
          <input
            style="text-align: right"
            type="text"
            id="address"
            class="form-control"
            placeholder="عنوان الشحن"
            v-model="searchAddress"
          />
          <span class="text-danger" v-if="errors && errors.address">
            {{ errors.address[0] }}
          </span>
        </div>

        <div class="map-container mb-3">
          <div id="map" style="height: 400px"></div>
        </div>
        <button
          type="submit"
          class="btn"
          style="background-color: var(--primary-color)"
        >
          <span v-if="loading" class="spinner-border spinner-border-sm"></span>
          <span v-if="!loading">التالى : تأكيد الطلب</span>
        </button>
      </form>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

// Define initMap function globally
function initMap() {
  // Initialize map
  this.map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: -34.397, lng: 150.644 },
    zoom: 8,
  });

  // Add click event listener to map
  this.map.addListener("click", (event) => {
    this.placeMarker(event.latLng);
  });
}
export default {
  computed: {
    ...mapGetters("auth", ["user", "loading", "errors"]),
    ...mapGetters("checkout", ["errors", "delivery_details"]),
  },
  data() {
    return {
      map: null,
      marker: null,
      searchAddress: "", // Holds the value of the address search input
      autocomplete: null, // Google Places Autocomplete object
    };
  },
  methods: {
    ...mapActions("checkout", ["storeAddress", "nextStep"]),
    initMap() {
      // Initialize map
      this.map = new google.maps.Map(document.getElementById("map"), {
        center: { lat: 26.8206, lng: 30.8025 },
        zoom: 9,
      });
      // Get user's current location using Geolocation API
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const userLocation = {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            };
            // Place marker on user's current location
            this.placeMarker(userLocation);
            // Center map on user's current location
            this.map.setCenter(userLocation);

            const geocoder = new google.maps.Geocoder();
            geocoder.geocode({ location: userLocation }, (results, status) => {
              if (status === "OK") {
                if (results[0]) {
                  this.searchAddress = results[0].formatted_address;
                }
              }
            });
          },
          (error) => {
            console.error("Error getting user's current location:", error);
          }
        );
      } else {
        console.error("Geolocation is not supported by this browser.");
      }

      // Initialize autocomplete
      this.autocomplete = new google.maps.places.Autocomplete(
        document.getElementById("address"),
        {
          fields: ["formatted_address", "geometry"],
          types: ["address"],
        }
      );

      // Add event listener for when a place is selected from autocomplete
      this.autocomplete.addListener("place_changed", () => {
        const place = this.autocomplete.getPlace();
        if (!place.geometry) {
          console.error("No details available for input: '" + place.name + "'");
          return;
        }

        // Update map marker and address field with selected place details
        this.placeMarker(place.geometry.location);
        // this.user.address = place.formatted_address;
        this.searchAddress = place.formatted_address;
        this.map.setCenter(place.geometry.location); // Center the map at the selected place
      });

      // Add click event listener to map
      this.map.addListener("click", (event) => {
        this.placeMarker(event.latLng);
      });
    },
    placeMarker(location) {
      // Place marker on map
      if (this.marker) {
        this.marker.setPosition(location);
      } else {
        this.marker = new google.maps.Marker({
          position: location,
          map: this.map,
        });
      }
      // Perform reverse geocoding to get address details based on the clicked location
      const geocoder = new google.maps.Geocoder();
      geocoder.geocode({ location: location }, (results, status) => {
        if (status === "OK") {
          if (results[0]) {
            console.log(results);
            // Iterate over results to find the most accurate address
            let mostAccurateAddress = results[0];
            for (let i = 1; i < results.length; i++) {
              if (results[i].geometry.location_type === "APPROXIMATE") {
                mostAccurateAddress = results[i];
                break; // Stop iterating once a rooftop address is found
              }
            }
            console.log(mostAccurateAddress.formatted_address)

            // Update address field with the formatted address of the most accurate result
            // this.user.address = mostAccurateAddress.formatted_address;
            this.searchAddress =  mostAccurateAddress.formatted_address;
          }
        }
      });
    },
    saveAddress() {
      let data = {
        name: this.user.name,
        phone: this.user.phone,
        email: this.user.phone,
        // email: this.user.email,
        // email: this.$store.getters["auth/isLoggedIn"]
        //   ? this.user.email
        //   : "guest@gmail.com",
        address: this.user.address,
      };
      // Check if the user is authenticated
      if (this.$store.getters["auth/isLoggedIn"]) {
        this.storeAddress(data).then(() => {
          this.nextStep();
        });
      } else {
        this.$store
          .dispatch("auth/registerGuestUser", data)
          .then(() => {
            this.storeAddress(data).then(() => {
              this.nextStep();
            });
          })
          .catch((error) => {
            console.error("Error checking email:", error);
            console.log(error.message);
            if (error.message === "Request failed with status code 422") {
              this.$awn.alert("رقم الهاتف موجود بالفعل");
            } else {
              this.$awn.alert("حدث خطأ أثناء فحص الهاتف.");
            }
          });
      }
    },
  },
  mounted() {
    // Load Google Maps API script dynamically
    const script = document.createElement("script");
    script.src = `https://maps.googleapis.com/maps/api/js?key=AIzaSyBUK88jmlcZv3IdJlhp944cJmzkWKelqq4&callback=initMap&libraries=places&callback=initMap`;
    script.defer = true;

    // Define the initMap function in the global scope
    window.initMap = this.initMap;

    document.head.appendChild(script);
  },
};
</script>
