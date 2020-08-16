<template>
  <v-app>
    <v-main>
      <Navbar
        :guessed="guessed"/>
      <v-container>
        <v-card
          outlined
          max-width="300px"
          class="mx-auto">
          <v-card-title>
            <span class="h6 mx-auto">Guess color by it's hex</span>
            <v-text-field
              class="mt-5"
              v-model="guessColor"
              color="grey darken-4"
              outlined
              disabled/>
          </v-card-title>
        </v-card>
        <v-card
          outlined
          max-width="700px"
          class="mt-4 px-3 mx-auto">
          <v-row
            v-for="(row, i) in colors"
            :key="i"
            justify="center"
            align="center">
            <v-col
              cols="4"
              v-for="(color, j) in row"
              :key="j"
              align-self="center">
              <Card
                @clicked-color="cliked"
                :color="color"/>
            </v-col>
          </v-row>
        </v-card>
        <v-container fluid>
          <v-btn
            fixed right bottom
            dark
            color="deep-orange darken-3"
            fab
            @click="skip">
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
          <v-btn
            fixed left bottom
            dark
            outlined
            color="deep-orange darken-3"
            fab
            @click="restart">
            <v-icon>mdi-restart</v-icon>
          </v-btn>
        </v-container>
      </v-container>
      <SuccessOverlay
        :success="success"
        :color="colorGuess"
        :name="colorName"
        @close="success = !success"/>
    </v-main>
  </v-app>
</template>

<script>
import Card from './components/Card.vue';
import Navbar from './components/Navbar.vue';
import SuccessOverlay from './components/SuccessOverlay.vue';

export default {
  name: 'App',
  components: {
    Card,
    Navbar,
    SuccessOverlay,
  },
  data: () => ({
    colors: [],
    success: false,
    colorGuess: '',
    colorName: '',
    guessed: 0,
  }),
  beforeMount() {
    this.genColors();
    if (!localStorage.guessed) localStorage.guessed = 0;
    else this.guessed = Number(localStorage.guessed);
  },
  computed: {
    guessColor() {
      return this.colors[Math.floor(Math.random() * 2)][Math.floor(Math.random() * 3)];
    },
  },
  methods: {
    async fetchColor(color) {
      const colorApi = `https://cors-anywhere.herokuapp.com/http://thecolorapi.com/id?hex=${color.slice(1)}&format=json`;
      let name = await fetch(colorApi, {
        headers: { Origin: 'http://thecolorapi.com/' },
      });
      name = await name.json();
      return name.name.value;
    },
    changeScore(isAdding = false) {
      const guessed = Number(localStorage.guessed);
      if (isAdding) {
        localStorage.guessed = guessed + 1;
        this.guessed = guessed + 1;
      } else {
        localStorage.guessed = guessed - 1;
        this.guessed = guessed - 1;
      }
    },
    async cliked(color) {
      const guess = this.guessColor;
      if (color === guess) {
        this.changeScore(true);
        this.success = !this.success;
        this.colorGuess = guess;
        this.colorName = await this.fetchColor(guess);
        setTimeout(() => {
          this.genColors();
          this.success = false;
          this.colorName = '';
        }, 3000);
      } else {
        this.changeScore();
      }
    },
    skip() {
      this.changeScore();
      this.genColors();
    },
    restart() {
      localStorage.guessed = 0;
      this.guessed = 0;
      this.genColors();
    },
    genColors() {
      const res = [];
      for (let i = 0; i < 2; i += 1) {
        const row = [];
        for (let j = 0; j < 3; j += 1) {
          // eslint-disable-next-line no-bitwise
          row.push(`#${(Math.random() * 0xFFFFFF << 0).toString(16).padStart(6, '0')}`);
        }
        res.push(row);
      }
      this.colors = res;
    },
  },
};
</script>
