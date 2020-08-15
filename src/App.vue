<template>
  <v-app>
    <v-main>
      <Navbar/>
      <v-scale-transition>
        <v-container>
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
        </v-container>
      </v-scale-transition>
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
  }),
  beforeMount() {
    this.genColors();
  },
  computed: {
    guessColor() {
      return this.colors[Math.floor(Math.random() * 2)][Math.floor(Math.random() * 3)];
    },
  },
  methods: {
    async cliked(color) {
      const guess = this.guessColor;
      const colorApi = `https://cors-anywhere.herokuapp.com/http://thecolorapi.com/id?hex=${guess.slice(1)}&format=json`;
      if (color === guess) {
        this.success = !this.success;
        this.colorGuess = guess;
        let name = await fetch(colorApi + guess.slice(1), {
          headers: { Origin: 'http://thecolorapi.com/' },
        });
        name = await name.json();
        this.colorName = name.name.value;
        setTimeout(() => {
          this.genColors();
          this.success = false;
          this.colorName = '';
        }, 2000);
      }
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
