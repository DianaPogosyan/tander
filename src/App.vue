<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <v-spacer></v-spacer>
    </v-app-bar>
    <v-main>
      <bitcoin/>
    </v-main>
  </v-app>
</template>

<style lang="scss">
  .btn-style {
    margin-bottom: 20px;
  }
  .fs {
    font-size: 1rem!important;
  }
</style>
<script>
import bitcoin from './components/bitcoin';

export default {
  name: 'App',

  components: {
    bitcoin,
  },

  data: () => ({
    //
  }),
};
</script>
