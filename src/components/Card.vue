<template>
  <v-card
    @click="handleClick(color)"
    height="7rem"
    max-width="450px"
    :color="color">
    <v-card-text>
      <div v-if="info" class="headline">
        <v-icon class="mb-1 mr-1" large>mdi-attachment</v-icon>
        <p v-if="guessedColor" style="display: inline-block">{{guessedColor}}</p>
      </div>
    </v-card-text>
  </v-card>
</template>

<script>
export default {
  name: 'Card',
  props: {
    color: {
      type: String,
      required: true,
    },
    guessedColor: {
      type: String,
      required: false,
      default: '',
    },
    info: {
      type: Boolean,
      required: false,
      default: false,
    },
    history: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  methods: {
    handleClick(color) {
      if (this.history) {
        this.$emit('copy-color', color);
      } else {
        this.$emit('clicked-color', color);
      }
    },
  },
};
</script>
