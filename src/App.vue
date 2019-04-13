<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>NAICS Visualization</span>
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
      <HierarchicalTreeView/>
    </v-content>
  </v-app>
</template>

<script>
import HierarchicalTreeView from './components/HierarchicalTreeView'

export default {
  name: 'App',
  components: {
    HierarchicalTreeView,
  },
  data () {
    return {
      //
    }
  }
}
</script>
