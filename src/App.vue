<template>
  <v-app>
    <Calculator />
  </v-app>
</template>

<script>
import Calculator from "./components/Calculator.vue";
export default {
  name: "App",

  components: {
    Calculator,
  },

  data: () => ({
    //
  }),
};
</script>
