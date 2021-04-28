<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
          <h1>Podcast search</h1>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <Search/>
    </v-main>
  </v-app>
</template>

<script>
import Search from './components/Search';

export default {
  name: 'App',

  components: {
    Search,
  },

  data: () => ({
    //
  }),
};
</script>
