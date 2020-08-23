<template>
  <v-card
    @click="handleClick(color)"
    height="7rem"
    max-width="450px"
    :color="color">
    <v-card-text>
      <p v-if="info" class="headline">
        <v-icon class="mb-1" large>mdi-attachment</v-icon>
        <!-- Guessed Color -->
      </p>
      <p />
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
