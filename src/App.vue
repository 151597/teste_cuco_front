<template>
  <v-app>
    <app-navigation></app-navigation>
    <v-main class="grey lighten-2">
      <v-container>
        <router-view/>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import SystemBar from './components/SystemBar.vue'
export default {
  name: 'App',
  components: {
    'app-navigation': SystemBar
  },
};
</script>
