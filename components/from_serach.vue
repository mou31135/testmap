<template>
  <div>
    <div class="col-12">
      <b-form>
        <b-form-group
          id="input-group-1"
          label="location:"
          label-for="input-1"
          description="We'll never share your location with anyone else."
        >
          <input
            class="form-control"
            id="searchTextField"
            v-model="location"
            ref="searchTextField"
            type="text"
          />
        </b-form-group>
      </b-form>
    </div>
    <div class="col-12">
      <div class="row">
        <div id="map" class="col-lg-auto"></div>
        <div id="listing" class="col-3">
          <table id="resultsTable">
            <tbody id="results"></tbody>
          </table>
        </div>
        <div style="display: none">
          <div id="info-content">
            <table>
              <tr id="iw-url-row" class="iw_table_row">
                <td id="iw-icon" class="iw_table_icon"></td>
                <td id="iw-url"></td>
              </tr>
              <tr id="iw-address-row" class="iw_table_row">
                <td class="iw_attribute_name">Address:</td>
                <td id="iw-address"></td>
              </tr>
              <tr id="iw-phone-row" class="iw_table_row">
                <td class="iw_attribute_name">Telephone:</td>
                <td id="iw-phone"></td>
              </tr>
              <tr id="iw-rating-row" class="iw_table_row">
                <td class="iw_attribute_name">Rating:</td>
                <td id="iw-rating"></td>
              </tr>
              <tr id="iw-website-row" class="iw_table_row">
                <td class="iw_attribute_name">Website:</td>
                <td id="iw-website"></td>
              </tr>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "@nuxtjs/axios";

let map;
let places;
let infoWindow;
let markers = [];
let autocomplete;
const api_key = "AIzaSyCz5WpsaHxLqy1qFCo4fOPVj-kkVDfcxhU";
const countryRestrict = { country: "us" };
const MARKER_PATH =
  "https://developers.google.com/maps/documentation/javascript/images/marker_green";
const hostnameRegexp = new RegExp("^https?://.+?/");

export default {
  head() {
    return {
      script: [
        {
          src:
            "https://maps.googleapis.com/maps/api/js?key=" +
            api_key +
            "&callback=initMap&libraries=places&v=weekly",
        },
      ],
    };
  },
  data: () => ({
    location: "",
    searchResults: [],
    arrSearchRes: [],
    defaultPlace: "ChIJX5dpCoGb4jARUE_iXbIAAQM",
    service: null, // will reveal this later 🕵️
  }),
  mounted() {
    this.initMap();
  },
  methods: {
    onPlaceChanged: function () {
    //----------------------onchange by  autocomplete method------------------------//

      const place = autocomplete.getPlace();

      if (place.geometry) {
        map.panTo(place.geometry.location);
        map.setZoom(15);
        this.search(place);
      } else {
        document.getElementById("searchTextField").placeholder = "Enter a city";
      }
    },
    initMap: function () {
      
      const vm = this;
      map = new google.maps.Map(document.getElementById("map"), {
        center: { lat: 13.812671, lng: 100.52151 },
        zoom: 14,
      });

      infoWindow = new google.maps.InfoWindow({
        content: document.getElementById("info-content"),
      });

      var input = this.$refs.searchTextField; //.getElementById('searchTextField');

      autocomplete = new google.maps.places.Autocomplete(input);
      places = new google.maps.places.PlacesService(map);
      autocomplete.addListener("place_changed", this.onPlaceChanged);
      //-------------------check input defult-----------------------------//
      if (input) {
        places.getDetails(
          { placeId: this.defaultPlace },
          function (place, status) {
            if (status !== google.maps.places.PlacesServiceStatus.OK) {
              return;
            }
             vm.search(place);

          }
        );
      }
    },
    clearMarkers: function () {
      //----------------------cler marker-------------------------//
      for (let i = 0; i < markers.length; i++) {
        if (markers[i]) {
          markers[i].setMap(null);
        }
      }
      markers = [];
    },
    showInfoWindow: function (data) {
      //----------------------call data restaurant-------------------------//

      const vm = this;
      const marker = data;

      places.getDetails(
        { placeId: marker.placeResult.place_id },
        function (place, status) {
          if (status !== google.maps.places.PlacesServiceStatus.OK) {
            return;
          }

          infoWindow.open(map, data);
          vm.buildIWContent(place);
        }
      );
    },
    search: function (place) {
      //----------------------search nearby data restaurant by place data-------------------------//

      const vm = this;

      const search = {
        bounds: map.getBounds(),
        types: ["restaurant"],
      };
      const location_plcae_id = place.place_id;
      console.info(location_plcae_id);
      places.nearbySearch(search, function (results, status) {
        if (status === google.maps.places.PlacesServiceStatus.OK) {
          vm.clearResults();
          vm.clearMarkers();

          // Create a marker for each hotel found, and
          // assign a letter of the alphabetic to each marker icon.
          for (let i = 0; i < results.length; i++) {
            const markerLetter = String.fromCharCode("A".charCodeAt(0) + (i % 26));
            const markerIcon = MARKER_PATH + markerLetter + ".png";
            // Use marker animation to drop the icons incrementally on the map.
            markers[i] = new google.maps.Marker({
              position: results[i].geometry.location,
              animation: google.maps.Animation.DROP,
              icon: markerIcon,
            });
            // If the user clicks a hotel marker, show the details of that hotel
            // in an info window.
            markers[i].placeResult = results[i];
            google.maps.event.addListener(markers[i], "click", (e) => {
              // function call not working
              vm.showInfoWindow(markers[i]);
            });

            setTimeout(vm.dropMarker(i), i * 100);
            vm.addResult(results[i], i);
          }
        }
      });
    },
    clearResults: function () {
     //----------------------clear right display data restaurant list------------------------//

      const results = document.getElementById("results");

      while (results.childNodes[0]) {
        results.removeChild(results.childNodes[0]);
      }
    },
    dropMarker: function (i) {
     //----------------------show display data restaurant marker------------------------//

      return function () {
        markers[i].setMap(map);
      };
    },
    addResult: function (result, i) {
     //----------------------show right display data restaurant list------------------------//

      const results = document.getElementById("results");
      const markerLetter = String.fromCharCode("A".charCodeAt(0) + (i % 26));
      const markerIcon = MARKER_PATH + markerLetter + ".png";
      const tr = document.createElement("tr");
      tr.style.backgroundColor = i % 2 === 0 ? "#F0F0F0" : "#FFFFFF";

      tr.onclick = function () {
        google.maps.event.trigger(markers[i], "click");
      };
      const iconTd = document.createElement("td");
      const nameTd = document.createElement("td");
      const icon = document.createElement("img");
      icon.src = markerIcon;
      icon.setAttribute("class", "placeIcon");
      icon.setAttribute("className", "placeIcon");
      const name = document.createTextNode(result.name);
      iconTd.appendChild(icon);
      nameTd.appendChild(name);
      tr.appendChild(iconTd);
      tr.appendChild(nameTd);
      results.appendChild(tr);
    },

    buildIWContent: function (place) {
    //----------------------show popup  data restaurant ------------------------//

      document.getElementById("iw-icon").innerHTML =
        '<img class="restaurantIcon" ' + 'src="' + place.icon + '"/>';
      document.getElementById("iw-url").innerHTML =
        '<b><a href="' + place.url + '">' + place.name + "</a></b>";
      document.getElementById("iw-address").textContent = place.vicinity;

      if (place.formatted_phone_number) {
        document.getElementById("iw-phone-row").style.display = "";
        document.getElementById("iw-phone").textContent = place.formatted_phone_number;
      } else {
        document.getElementById("iw-phone-row").style.display = "none";
      }

      // Assign a five-star rating to the hotel, using a black star ('&#10029;')
      // to indicate the rating the hotel has earned, and a white star ('&#10025;')
      // for the rating points not achieved.
      if (place.rating) {
        let ratingHtml = "";

        for (let i = 0; i < 5; i++) {
          if (place.rating < i + 0.5) {
            ratingHtml += "&#10025;";
          } else {
            ratingHtml += "&#10029;";
          }
          document.getElementById("iw-rating-row").style.display = "";
          document.getElementById("iw-rating").innerHTML = ratingHtml;
        }
      } else {
        document.getElementById("iw-rating-row").style.display = "none";
      }

      // The regexp isolates the first part of the URL (domain plus subdomain)
      // to give a short URL for displaying in the info window.
      if (place.website) {
        let fullUrl = place.website;
        let website = String(hostnameRegexp.exec(place.website));

        if (!website) {
          website = "http://" + place.website + "/";
          fullUrl = website;
        }
        document.getElementById("iw-website-row").style.display = "";
        document.getElementById("iw-website").textContent = website;
      } else {
        document.getElementById("iw-website-row").style.display = "none";
      }
    },
  },
};
</script>

<style>
#map {
  height: 800px;
  width: 800px;
}

#input-group-1 {
  width: 60%;
}

form {
  text-align: -webkit-center;
}

.placeIcon {
  cursor: pointer;
}
</style>
