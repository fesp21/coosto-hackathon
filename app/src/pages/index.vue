<template>
  <section class="container">

    <gmap-map
      :center="center"
      :zoom="12"
      style="width: 100%; height: 500px"
      v-if="shouldRender"
    >
      <gmap-info-window
        v-if="infoWindowLocation"
        :options="infoOptions"
        :position="infoWindowPos"
        :opened="infoWinOpen"
        @closeclick="infoWinOpen=false"
      >
        <router-link :to="{ name: 'location-id', params: { id: infoWindowLocation.id }}">
          {{ infoWindowLocation.name }}
        </router-link>
      </gmap-info-window>

      <gmap-marker
        v-for="(m, index) in places"
        :position="m.location"
        :draggable="true"
        :clickable="true"
        @click="toggleInfoWindow(m, index)"
      ></gmap-marker>
    </gmap-map>

  </section>
</template>

<script>
import apollo from '~/lib/apollo'
import { mapDefaults } from '~/lib/const'
import gql from 'graphql-tag'

export default {
  async asyncData () {
    const { data } = await apollo.query({
      query: gql`{
        dogPlaces {
          id, name, location { lat, lng }
        }
      }`
    })

    return {
      ...mapDefaults,
      shouldRender: false,
      // records: res.data.records,
      center: { lat: 51.4416, lng: 5.4697 },
      places: data.dogPlaces
    }
  },

  computed: {
    renderPanorama () {
      return this.infoWindowPos && this.infoWindowPos.lat && this.infoWindowPos.lng
    }
  },

  methods: {
    toggleInfoWindow: function (marker, idx) {
      this.infoWindowPos = marker.location
      this.infoWindowLocation = marker
      // check if its the same marker that was selected if yes toggle
      if (this.currentMidx === idx) {
        this.infoWinOpen = !this.infoWinOpen
      } else {
        // if different marker set infowindow to open and reset current marker index
        this.infoWinOpen = true
        this.currentMidx = idx
      }
    }
  },

  mounted () {
    this.shouldRender = true
  }
}
</script>


<style scoped>
.title {
  margin: 50px 0;
}
</style>
