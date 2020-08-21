<template>
  <v-container fluid>
    <v-btn
      fab
      x-large
      :dark="darkMode"
      @click="$emit('toggle-dark-mode')"
      fixed bottom left>
      <v-icon large>
        {{darkMode ? 'mdi-white-balance-sunny'
            : 'mdi-weather-night'}}
      </v-icon>
    </v-btn>
    <v-row>
      <v-col
        sm="2"
        cols="5"
        class="ml-auto">
        <v-btn-toggle
          rounded
          v-model="toggle"
          :dark="darkMode"
          class="fixed-bottom-right">
          <v-btn
            fab
            large
            outlined
            elevation="4"
            :dark="darkMode"
            @click="click(true)">
            <v-icon>mdi-restart</v-icon>
          </v-btn>
          <v-btn
            fab
            large
            outlined
            elevation="4"
            :dark="darkMode"
            @click="click">
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
        </v-btn-toggle>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'ButonsGroup',
  props: {
    darkMode: {
      type: Boolean,
      required: true,
    },
  },
  watch: {
    toggle(val) {
      console.log(val);
    },
  },
  data() {
    return {
      toggle: undefined,
    };
  },
  methods: {
    click(restart = false) {
      if (restart) this.$emit('restart');
      else this.$emit('restart');
      setInterval(() => { this.toggle = undefined; }, 750);
    },
  },
};
</script>

<style scoped>
.fixed-bottom-right {
  position: fixed;
  bottom: 2.5%;
  right: 3%;
}
</style>
