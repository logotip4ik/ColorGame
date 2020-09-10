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
        <v-list v-for="(date, i) in getColorDates" :key="i">
          <v-subheader class="headline">{{new Date(date).toDateString()}}</v-subheader>
          <v-divider />
          <v-list-item v-for="color in getColorsByDate[i]" :key="color.id">
            <v-container>
              <v-card
                flat
                class="mx-auto"
                max-width="1000">
                <v-list-item-title
                  class="headline mb-2"
                  style="font-size: 1.3rem!important;">
                  {{color.time.substring(0, 5)}}
                </v-list-item-title>
                <v-card flat>
                  <v-row
                    v-for="(row, I) in color.colors"
                    :key="I"
                    justify="center"
                    align="center">
                    <v-col
                      cols="4"
                      v-for="(col, j) in row"
                      :key="j"
                      align-self="center">
                      <v-fade-transition>
                        <Card
                          history
                          :color="col"
                          :guessedColor="color.guessedColorName"
                          class="mx-auto"
                          @copy-color="copyColor"
                          :info="col === color.guessedColor"/>
                      </v-fade-transition>
                    </v-col>
                  </v-row>
                </v-card>
              <v-divider />
              </v-card>
            </v-container>
          </v-list-item>
        </v-list>
      </v-container>
    </v-card>
    <ReUsableSnackBar
      :text="text"
      :show="showSnack"
      @close="showSnack = !showSnack"/>
  </v-dialog>
</template>

<script>
import Card from './Card.vue';
import ReUsableSnackBar from './ReUsableSnackBar.vue';

export default {
  name: 'HistoryOverlay',
  components: {
    Card,
    ReUsableSnackBar,
  },
  data() {
    return {
      showSnack: false,
      text: '',
    };
  },
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
      required: true,
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
    sortedColors() {
      // eslint-disable-next-line vue/no-side-effects-in-computed-properties
      return this.colors.sort((a, b) => {
        const aTime = `${a.date} ${a.time}`;
        const bTime = `${b.date} ${b.time}`;
        return new Date(aTime) - new Date(bTime);
      });
    },
    getColorDates() {
      if (!this.sortedColors) return [];
      const res = new Set();
      for (let i = 0; i < this.sortedColors.length; i += 1) {
        const date = this.sortedColors[i]?.date;
        res.add(date);
      }
      return Array.from(res);
    },
    getColorsByDate() {
      const res = [];
      const dates = this.getColorDates;
      for (let i = 0; i < dates.length; i += 1) {
        const dateArr = [];
        this.sortedColors.forEach((color) => {
          dateArr.push(color);
        });
        res.push(dateArr);
      }
      return res;
    },
  },
  methods: {
    async copyColor(color) {
      if (!navigator.clipboard) {
        this.text = 'Can not copy to clipboard';
        this.showSnack = true;
        return;
      }
      await navigator.clipboard.writeText(color)
        .then(() => {
          this.text = `Copied ${color}`;
          this.showSnack = true;
        }).catch((err) => {
          this.text = err.message;
          this.showSnack = true;
        });
    },
  },
};
</script>
