<template>
  <v-snackbar
    v-model="nopeBool"
    timeout="2000">
  Nope this was color: {{nopeColor}}
  <template v-slot:action="{ attrs }">
    <v-btn
      color="white"
      text
      v-bind="attrs"
      small
      @click="$emit('close')"
    >
      <v-icon>mdi-close</v-icon>
    </v-btn>
  </template>
  </v-snackbar>
</template>

<script>
export default {
  name: 'SnackBar',
  props: {
    nope: {
      type: Boolean,
      required: true,
    },
    nopeColor: {
      type: String,
      required: true,
    },
  },
  computed: {
    nopeBool: {
      get() {
        return this.nope;
      },
      set() {
        this.$emit('close');
      },
    },
  },
};
</script>
