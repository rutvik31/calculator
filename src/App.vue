<template>
  <v-app>
    <v-app-bar elevation="0" v-if="$vuetify.breakpoint.smAndDown">
      <v-spacer></v-spacer>
      <v-btn class="float-right" icon @click="panel = !panel">
        <v-icon>mdi-menu</v-icon>
      </v-btn>
    </v-app-bar>
    <v-main class="backgroud d-flex align-end">
      <v-container fluid>
        <Calculator @refereshHistory="refereshHistory" />
      </v-container>
    </v-main>
    <v-navigation-drawer
      mobile-breakpoint="768"
      fixed
      right
      app
      class="backgroud"
      v-model="panel"
    >
      <v-list>
        <v-list-item>
          <v-list-item-title>History</v-list-item-title>
          <v-tooltip left nudge-left="0">
            <template v-slot:activator="{ on, attrs }">
              <v-icon @click="clearHistory()" v-bind="attrs" v-on="on">mdi-delete</v-icon>
            </template>
            <span>Clear all History</span>
          </v-tooltip>
        </v-list-item>
        <v-list-item v-for="(entry, index) in history" :key="index">
          <v-list-item-title>{{ entry.expression }}</v-list-item-title>
          <v-list-item-subtitle>{{ entry.result }}</v-list-item-subtitle>
        </v-list-item>
        <v-list-item v-if="!history.length"
          >There is no history yet</v-list-item
        >
      </v-list>
    </v-navigation-drawer>
  </v-app>
</template>

<script>
import Calculator from "./components/Calculator.vue";
export default {
  name: "App",
  components: {
    Calculator,
  },
  data() {
    return {
      history: [],
      panel: this.$vuetify.breakpoint.smAndUp,
    };
  },
  methods: {
    refereshHistory() {
      this.history = JSON.parse(localStorage.getItem("history")) || [];
    },
    clearHistory() {
      localStorage.removeItem("history");
      this.refereshHistory();
    },
  },
  mounted() {
    this.refereshHistory();
  },
};
</script>
<style>
.backgroud {
  background-color: whitesmoke !important;
}
</style>
