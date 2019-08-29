<template>
  <v-app>
    <v-content>
      <Calculator />
    </v-content>
  </v-app>
</template>

<script>
import Calculator from "./components/Calculator";

export default {
  name: "App",
  meta: {
    title: "Vue+GoLang Full Stack Calculator"
  },
  components: {
    Calculator
  },
  data: () => ({})
};
</script>

<style lang="scss">
</style>