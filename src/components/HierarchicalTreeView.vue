<template>
   <v-card
    class="mx-auto"
    max-width="700"
  >
    <v-sheet class="pa-3 primary lighten-2">
      <v-text-field
        v-model="search"
        label="Search Industry Directory by Name"
        dark
        flat
        solo-inverted
        hide-details
        clearable
        clear-icon="mdi-close-circle-outline"
      ></v-text-field>
      <v-checkbox
        v-model="caseSensitive"
        dark
        hide-details
        label="Case sensitive search (NAICS code on the right)"
      ></v-checkbox>
    </v-sheet>
    <v-card-text>
      <v-treeview
        :items="items"
        :search="search"
        :filter="filter"
        :open.sync="open"
      >
        <template v-slot:prepend="{ item }">
          <v-icon
            v-if="item.children"
            v-text="`mdi-${item.id === 1 ? 'home-variant' : 'folder-network'}`"
          ></v-icon>
        </template>
        <template v-slot:label="{ item }">
          {{item.name | trim}}
        </template>
        <template slot="append" slot-scope="{ item }">
            {{item.id}}
        </template>
      </v-treeview>
    </v-card-text>
  </v-card>
</template>

<script>
  import IndustriesTree from '../data/graph.json'
// src/data/graph.json
  export default {
    data: () => ({
      open: [1, 2],
      search: null,
      caseSensitive: false,
      items: [],
    }),
    computed: {
      filter () {
        return this.caseSensitive
          ? (item, search, textKey) => item[textKey].indexOf(search) > -1 : undefined;
      }
    },
    filters: {
      trim: (name) => {
        const cutOff = 38;
        if (name.length > cutOff) {
          return name.slice(0, cutOff);
        } else {
          return name;
        }
      }
    },
    created() {
      let i;
      for (i = 0; i < IndustriesTree.children.length; i++) {
        this.items.push({
          id: IndustriesTree.children[i].id,
          name: IndustriesTree.children[i].name,
          children: IndustriesTree.children[i].children,
        })
      }
    }
  }
</script>
