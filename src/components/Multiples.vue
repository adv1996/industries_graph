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
          color="white"
          tile
          height="400"
        >
          <v-card-title primary-title>
            <div>
              <h3 v-if="rows[category][0][3].length < 45">{{ rows[category][0][3].replace('and', '&') }}</h3>
              <h3 v-else>{{ rows[category][0][3].slice(0, 45).replace('and', '&') + '...' }}</h3>
            </div>
          </v-card-title>
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
      },
    }
  }
</script>

<style lang="scss" scoped>

</style>