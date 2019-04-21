<template>
  <v-app>
    <v-toolbar app color="light-blue">
      <v-toolbar-title class="headline text-uppercase">
        <span>NAICS Viz</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <a :href="github" target="_blank"> 
        <v-btn
          color="black"
          flat
        >
          <font-awesome-icon :icon="['fab', 'github']" size="lg"/>
        </v-btn>
      </a>
    </v-toolbar>

    <v-content>
      <Multiples/>
      <HierarchicalTreeView/>
    </v-content>
  </v-app>
</template>

<script>
import Multiples from './components/Multiples'
import HierarchicalTreeView from './components/HierarchicalTreeView'

export default {
  name: 'App',
  components: {
    Multiples,
    HierarchicalTreeView
  },
  data () {
    return {
      github: 'https://github.com/adv1996/industries_graph',
    }
  }
}
</script>
