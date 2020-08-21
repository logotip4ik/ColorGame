<template>
  <v-dialog
    fullscreen
    :dark="dark"
    transition="dialog-bottom-transition"
    :value="showHistory">
    <v-toolbar>
      <v-toolbar-title>History</v-toolbar-title>
      <v-spacer />
      <v-toolbar-items>
        <v-btn
          text
          :dark="dark"
          @click="$emit('close')">
          <v-icon>
            mdi-close
          </v-icon>
        </v-btn>
      </v-toolbar-items>
    </v-toolbar>
    <v-card
      :dark="dark">
      <v-container>
        {{colorDates}}
      </v-container>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: 'HistoryOverlay',
  props: {
    show: {
      type: Boolean,
      required: true,
    },
    dark: {
      type: Boolean,
      required: true,
    },
    colors: {
      type: Array,
      // required: true,
    },
  },
  computed: {
    showHistory: {
      get() {
        return this.show;
      },
      set() {
        this.$emit('close');
      },
    },
    colorDates() {
      if (!this.colors) return [];
      const res = [];
      for (let i = 0; i < this.colors.length; i += 1) {
        const color = this.colors[i];
        res.push(color);
      }
      return res;
    },
  },
};
</script>
