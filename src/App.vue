<template>
  <v-app
    :dark="dark"
    :class="['opacity-transition', dark ? 'dark-background' : ''].join(' ')">
    <v-main>
      <Navbar
        :guessed="guessed"/>
      <v-container>
        <v-card
          :dark="dark"
          outlined
          :color=" dark ? 'grey darken-4' : ''"
          max-width="300px"
          class="mx-auto opacity-transition">
          <v-card-title>
            <span class="h6 mx-auto">Guess color by it's hex</span>
            <v-text-field
              :dark="dark"
              class="mt-5 dark-input"
              v-model="guessColor"
              outlined
              disabled/>
          </v-card-title>
        </v-card>
        <v-card
          outlined
          :dark="dark"
          max-width="700px"
          :color=" dark ? 'grey darken-4' : ''"
          class="mt-4 px-3 mx-auto opacity-transition">
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
              <v-fade-transition>
                <Card
                  @clicked-color="cliked"
                  :color="color"/>
              </v-fade-transition>
            </v-col>
          </v-row>
        </v-card>
      </v-container>
      <ButtonsGroup
        :darkMode="dark"
        @skip="skip"
        @restart="restart"
        @toggle-dark-mode="toggleDarkMode"/>
      <SuccessOverlay
        :loading="loading"
        :success="success"
        :color="colorGuess"
        :name="colorName"
        @close="success = !success"/>
      <SnackBar
        :nope="nope"
        :nopeColor="nopeColor"
        @close="closeSnackBar"/>
      <NoNetworkSnackBar
        :noInternet="noNetwork"
        @close="network = false"/>
    </v-main>
  </v-app>
</template>

<script>
import Card from './components/Card.vue';
import Navbar from './components/Navbar.vue';
import SnackBar from './components/SnackBar.vue';
import ButtonsGroup from './components/ButtonsGroup.vue';
import SuccessOverlay from './components/SuccessOverlay.vue';
import NoNetworkSnackBar from './components/NoNetworkSnackBar.vue';

export default {
  name: 'App',
  components: {
    Card,
    Navbar,
    SnackBar,
    ButtonsGroup,
    SuccessOverlay,
    NoNetworkSnackBar,
  },
  data: () => ({
    colors: [],
    loading: false,
    success: false,
    firstTry: true,
    noNetwork: false,
    dark: false,
    nope: false,
    nopeColor: '',
    colorGuess: '',
    colorName: '',
    guessed: 0,
  }),
  beforeMount() {
    this.genColors();
    if (!localStorage.guessed) localStorage.guessed = 0;
    else this.guessed = Number(localStorage.guessed);
    if (!localStorage.darkMode) localStorage.darkMode = false;
    else this.dark = !!JSON.parse(localStorage.darkMode);
  },
  computed: {
    guessColor() {
      return this.colors[Math.floor(Math.random() * 2)][Math.floor(Math.random() * 3)];
    },
  },
  methods: {
    async fetchColor(color) {
      const colorApi = `https://cors-anywhere.herokuapp.com/http://thecolorapi.com/id?hex=${color.slice(1)}`;
      try {
        let name = await fetch(colorApi, {
          headers: {
            Origin: 'http://thecolorapi.com/',
            'Access-Control-Allow-Origin': '*',
          },
        });
        name = await name.json();
        return name.name.value;
      } catch (error) {
        return undefined;
      }
    },
    changeScore(isAdding = false) {
      if (this.firstTry) {
        this.firstTry = !this.firstTry;
        return;
      }
      const guessed = Number(localStorage.guessed);
      if (isAdding) {
        localStorage.guessed = guessed + 1;
        this.guessed = guessed + 1;
      } else {
        if (guessed <= 0) return;
        localStorage.guessed = guessed - 1;
        this.guessed = guessed - 1;
      }
    },
    async cliked(color) {
      const guess = this.guessColor;
      if (color === guess) {
        this.changeScore(true);
        this.nope = false;
        this.firstTry = !this.firstTry;
        this.success = !this.success;
        this.colorGuess = guess;
        this.loading = !this.loading;
        // this.colorName = await this.fetchColor(guess);
        let colorName;
        if (!this.noInternet) {
          colorName = await this.fetchColor(guess);
        }
        if (!colorName) {
          this.noNetwork = true;
          this.colorName = 'No Internet Connection';
        }
        setTimeout(() => {
          this.genColors();
          this.success = false;
          this.colorName = '';
        }, 3000);
      } else {
        this.changeScore();
        this.nope = true;
        this.nopeColor = color;
      }
      this.loading = false;
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
    closeSnackBar() {
      this.nope = false;
      this.nopeColor = '';
    },
    toggleDarkMode() {
      localStorage.darkMode = !this.dark;
      this.dark = !this.dark;
    },
  },
};
</script>

<style scoped>
/* .dark-input .v-input__control .v-input__slot .v-text-field__slot input:disabled{
  color: black!important;
} */
.opacity-transition {
  transition: all 0.5s!important;
}
.dark-background {
  background-color: black!important;
}
</style>
