<template>
  <v-container fluid grid-list-sm>
    <v-layout row wrap>
      <v-flex
        xs12
        md4
        v-for="category in categories"
        :key="category"
        d-flex
      >
        <v-card
          color="blue lighten-4"
          tile
          height="350"
        >
          <v-card-text>{{ rows[category][0][3] }}</v-card-text>
          <Map :data="rows[category]" :category="category"/>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  import _ from 'lodash';
  import IndustryData from '../data/naics.json';
  import Map from './Map.vue';
  
  export default {
    components: {
      Map,
    },
    data() {
      return {
        rows: [],
        categories: [],
      }
    },
    created() {
      this.setData();
    },
    methods: {
      setData() {
        this.rows = _.groupBy(IndustryData.rows, (r) => {
          return r[3][0] + r[2]
        })
        this.categories = Object.keys(this.rows)
      }
    }
  }
</script>

<style lang="scss" scoped>

</style>