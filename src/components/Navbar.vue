// deep-orange darken-3

<template>
  <v-app-bar
    dense
    dark
    color="grey darken-4">
    <v-toolbar-title>
      Color Game
    </v-toolbar-title>
    <v-spacer/>
    <v-toolbar-items>
      <!-- <h2 @click="">{{colorGuessed}}</h2> -->
      <v-tooltip
        bottom
        nudge-bottom="15px"
        :open-on-click="true"
        :open-on-focus="true">
        <template v-slot:activator="{ on, attrs }">
          <h3
            class="my-auto mr-4"
            v-bind="attrs"
            v-on="on">
            {{guessed}}</h3>
        </template>
        Number you guessed color. Each time you skip or
         you don't guess color your score will be subtracted
      </v-tooltip>
    </v-toolbar-items>
  </v-app-bar>
</template>

<script>
export default {
  name: 'Navbar',
  props: {
    guessed: {
      type: Number,
      required: true,
    },
  },
};
</script>
