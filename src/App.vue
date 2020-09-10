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
                  @clicked-color="clicked"
                  :color="color"/>
              </v-fade-transition>
            </v-col>
          </v-row>
        </v-card>
      </v-container>
      <v-overlay :value="loadingPage" :dark="dark">
        <v-progress-circular indeterminate size="64" />
      </v-overlay>
      <ButtonsGroup
        :darkMode="dark"
        @skip="skip"
        @restart="restart"
        @history="showHistory = !showHistory"
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
      <ReUsableSnackBar
        :show="error"
        :text="errorText"
        @close="network = false"/>
      <HistoryOverlay
        :dark="dark"
        :colors="history"
        :show="showHistory"
        @close="showHistory = !showHistory"/>
    </v-main>
  </v-app>
</template>

<script>
import { openDB } from 'idb';
import { v4 } from 'uuid';

import Card from './components/Card.vue';
import Navbar from './components/Navbar.vue';
import SnackBar from './components/SnackBar.vue';
import ButtonsGroup from './components/ButtonsGroup.vue';
import HistoryOverlay from './components/HistoryOverlay.vue';
import SuccessOverlay from './components/SuccessOverlay.vue';
import ReUsableSnackBar from './components/ReUsableSnackBar.vue';

export default {
  name: 'App',
  components: {
    Card,
    Navbar,
    SnackBar,
    ButtonsGroup,
    HistoryOverlay,
    SuccessOverlay,
    ReUsableSnackBar,
  },
  data: () => ({
    colors: [],
    history: [],
    loading: false,
    loadingPage: false,
    success: false,
    firstTry: true,
    showHistory: false,
    error: false,
    dark: false,
    nope: false,
    db: null,
    prevId: null,
    nopeColor: '',
    errorText: '',
    colorGuess: '',
    colorName: '',
    guessed: 0,
    id: '',
  }),
  async beforeMount() {
    await this.prepare();
  },
  computed: {
    guessColor() {
      return this.colors.length > 0
        ? this.colors[Math.floor(Math.random() * 2)][Math.floor(Math.random() * 3)]
        : '#FFFFFF';
    },
  },
  watch: {
    async guessColor() {
      this.id = v4();
      await this.getAllColorHistory();
    },
  },
  methods: {
    async prepare() {
      this.loadingPage = true;
      await this.CheckDB();
      await this.genColors();
      if (!localStorage.guessed) localStorage.guessed = 0;
      else this.guessed = Number(localStorage.guessed);
      if (!localStorage.darkMode) localStorage.darkMode = false;
      else this.dark = !!JSON.parse(localStorage.darkMode);
      this.loadingPage = false;
    },
    async fetchColor(color) {
      const colorApi = `https://cors-anywhere.herokuapp.com/http://thecolorapi.com/id?hex=${color.slice(1)}`;
      try {
        let name = await fetch(colorApi, {
          headers: {
            Origin: process.env.HOST || 'http://localhost:8080/',
            'Access-Control-Allow-Origin': '*',
          },
        });
        name = await name.json();
        return name.name.value;
      } catch (error) {
        console.log(error);
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
    async clicked(color) {
      const guess = this.guessColor;
      if (color === guess) {
        this.changeScore(true);
        this.nope = false;
        this.firstTry = !this.firstTry;
        this.success = !this.success;
        this.colorGuess = guess;
        this.loading = true;
        if (!this.error) {
          this.colorName = await this.fetchColor(guess);
        }
        if (!this.colorName) {
          this.error = true;
          this.errorText = 'No Internet Connection';
          this.colorName = 'No Internet Connection';
        }
        setTimeout(async () => {
          await this.addColorsToHistory(true, this.colorName);
          await this.genColors();
          this.success = false;
          this.colorName = '';
        }, 2000);
      } else {
        this.changeScore();
        this.nope = true;
        this.nopeColor = color;
      }
      this.loading = false;
    },
    async skip() {
      this.changeScore();
      await this.genColors();
    },
    restart() {
      localStorage.guessed = 0;
      this.guessed = 0;
      this.genColors();
    },
    async genColors() {
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
      await this.addColorsToHistory();
    },
    closeSnackBar() {
      this.nope = false;
      this.nopeColor = '';
    },
    toggleDarkMode() {
      localStorage.darkMode = !this.dark;
      this.dark = !this.dark;
    },
    async addColorsToHistory(guessed = false, name = null) {
      const date = new Date();
      const hours = date.getHours().toString().length === 1 ? `0${date.getHours()}` : date.getHours();
      const minutes = date.getMinutes().toString().length === 1 ? `0${date.getMinutes()}` : date.getMinutes();
      const seconds = date.getSeconds().toString().length === 1 ? `0${date.getSeconds()}` : date.getSeconds();
      const time = `${hours}:${minutes}:${seconds}`;
      const id = v4();
      const dataColors = {
        colors: this.colors,
        guessedColor: guessed ? this.guessColor : null,
        guessedColorName: name,
        date: date.toISOString().substring(0, 10),
        time,
        id: guessed ? this.prevId : id,
      };
      const store = this.db.transaction('history', 'readwrite').objectStore('history');
      store.put(dataColors);
      this.prevId = id;
    },
    async getAllColorHistory() {
      const store = this.db.transaction('history', 'readonly').objectStore('history');
      this.history = await store.getAll();
    },
    async CheckDB() {
      try {
        const db = await openDB('colorHistory', 10, {
          upgrade(dataBase) {
            if (dataBase.objectStoreNames[0] !== 'history') {
              dataBase.createObjectStore('history', {
                keyPath: 'id',
              });
            }
          },
        });
        this.db = db;
      } catch (error) {
        this.test = error;
      }
    },
  },
};
</script>

<style scoped>
.opacity-transition {
  transition: all 0.5s!important;
}
.dark-background {
  background-color: black!important;
}
</style>
