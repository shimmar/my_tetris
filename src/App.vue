<template>
  <v-app>
    <v-main>
      <TetrisField/>
    </v-main>
  </v-app>
</template>

<script>
import TetrisField from './components/TetrisField';

export default {
  name: 'App',

  components: {
    TetrisField
  },

  data: () => ({
    //
  }),
};
</script>
