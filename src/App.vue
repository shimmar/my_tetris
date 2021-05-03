<template>
  <v-app>
    <v-main>
      <GameField/>
    </v-main>
  </v-app>
</template>

<script>
import GameField from './components/GameField';

export default {
  name: 'App',

  components: {
    GameField
  },

  data: () => ({
    //
  }),
};
</script>
