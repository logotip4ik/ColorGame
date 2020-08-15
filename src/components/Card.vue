<template>
  <v-card
    @click="$emit('clicked-color', color)"
    height="10rem"
    max-width="450px"
    :color="color">
    <v-card-text>
      <p />
    </v-card-text>
  </v-card>
</template>

<script>
export default {
  name: 'Card',
  props: {
    color: String,
  },
};
</script>
