<template>
  <div id="booking-2">
    <h2>Where are we coming to clean?</h2>
    <h4>Add on extras for a cleaning upgrade.</h4>
    <gmap-autocomplete
        class="item button address"
        :class="{success: data.address, error: data.address === 'Address is not valid'}"
        :value="data.address"
        @place_changed="getAddressData($event)"
    >
    </gmap-autocomplete>
    <input class="item button apt" placeholder="APT">
    <input class="item button zip" placeholder="ZIP" readonly :value="data.zip">
    <div class="entrancesHolder">
      <v-select
          class="item button"
          label="item"
          label-option="item"
          :options="entrances"
          v-model="data.entrance"
          style2
          v-if="entrances.length"
      ></v-select>
    </div>
    <h2>How frequent?</h2>
    <h5 class="title">RECURRING</h5>
    <div
        class="item button frequent"
        :class="{active: item.text === data.frequent.text}"
        v-for="item in frequents"
        :key="item.text"
        @click="data.frequent = item"
    >
      <p class="name">{{item.text}}</p>
      <p class="sale" v-if="item.sale > 0">{{item.sale}}% off</p>
    </div>
    <div class="buttons">
      <a
          @click="$router.go(-1)"
          class="back"
      >
        < Back
      </a>
      <router-link
          :to="{name: 'Checkout'}"
          class="button active"
          tag="button"
          :disabled="data.address === 'Address is not valid' || !data.address"
      >
        Next
      </router-link>
    </div>
  </div>
</template>

<script>
import VSelect from "@/components/VSelect";
import {mask} from 'vue-the-mask'
import {gmapApi} from 'vue2-google-maps'

export default {
  name: "Booking-2",
  components: {
    VSelect
  },
  directives: {
    mask
  },
  data() {
    return {

    }
  },
  computed: {
    data() {
      return this.$store.state.dataToSend;
    },
    entrances() {
      return this.$store.state.entrances;
    },
    frequents() {
      return this.$store.state.frequents;
    },
    location() {
      return this.$store.state.location;
    },
    google: gmapApi
  },
  methods: {
    getAddressData(e) {
      console.log(e);
      let th = this
      new google.maps.Geocoder().geocode({'address': th.location.centeraddress}, (results, status) => {
        if (status === 'OK' && th.google.maps.geometry.spherical.computeDistanceBetween(e.geometry.location, results[0].geometry.location) < th.location.radius) {
          let zip = e.address_components.filter(item => item.types[0] === 'postal_code')[0]
          if (zip) {
            let address =  e.address_components.filter(item => item.types[0] === "street_number")[0].long_name + ' ' + e.address_components.filter(item => item.types[0] === "route")[0].long_name;
            th.$store.commit('dataToSend', {key: 'address', payload: address})
            th.$store.commit('dataToSend', {key: 'zip', payload: zip.long_name})
            this.$store.dispatch('setCache')
          } else {
            th.$store.commit('dataToSend', {key: 'address', payload: 'Address is not valid'})
          }
        } else {
          th.$store.commit('dataToSend', {key: 'address', payload: 'Address is not valid'})
        }
      })
    }
  },
  mounted() {
  }
}
</script>

<style lang="scss">
  #booking-2 {
    h4 {
      margin-top: vwD(20);
      margin-bottom: vwD(40);
    }
    h5 {
      width: 100%;
    }
    .item {
      padding: 0 vwD(25);
      position: relative;
    }
    .frequent {
      width: vwD(275);
      .sale {
        position: absolute;
        top: calc(100% + #{vwD(10)});
        left: 0;
        right: 0;
        margin: auto;
        text-align: center;
        font-size: vwD(13);
        font-weight: 600;
        opacity: .7;
        color: $color;
      }
    }
    input {
      cursor: auto;
      &:hover {
        background: none;
      }
      &[readonly] {
        background: #F0F0F0;
      }
    }
    .address {
      width: calc(50% - (#{vwD(20)} * 2 / 3));
      &.success {
        border-color: #00ff00;
      }
      &.error {
        border-color: red;
      }
    }
    .apt, .zip {
      width: calc(25% - (#{vwD(20)} * 2 / 3));
    }
    .entrancesHolder {
      margin: vwD(40) 0 vwD(45) 0;
      width: vwD(410);
    }
    &>.title {
      margin-top: vwD(40);
      margin-bottom: vwD(15);
    }
  }
  @media screen and (max-width: $mobileOn) {
    #booking-2 {
      h4 {
        margin-bottom: vwM(15);
      }
      h5.title {
        margin-bottom: vwM(12);
      }
      .item {
        padding: 0 vwM(25);
        margin-bottom: vwM(10);
      }
      .address, .apt, .zip, .entrancesHolder {
        width: 100%;
      }
      .frequent {
        width: 100%;
        justify-content: space-between;
        .sale {
          position: static;
          font-size: vwM(14);
          margin-right: 0;
          color: currentColor;
        }
      }
    }
  }
</style>
