<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>NAICS Viz</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        flat
        href="https://github.com/adv1996/industries_graph"
        target="_blank"
      >
        <span class="mr-2">Latest Release</span>
      </v-btn>
    </v-toolbar>

    <v-content>
      <Multiples/>
    </v-content>
  </v-app>
</template>

<script>
// import Map from './components/Map'
import Multiples from './components/Multiples'
export default {
  name: 'App',
  components: {
    Multiples
  },
  data () {
    return {
      //
    }
  }
}
</script>
