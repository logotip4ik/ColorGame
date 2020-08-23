<template>
  <v-snackbar
    v-model="showSnack"
    :timeout="timeout"
  >
    {{text}}
  </v-snackbar>
</template>

<script>
export default {
  name: 'ReUsableSnackBar',
  props: {
    show: {
      type: Boolean,
      required: true,
    },
    text: {
      type: String,
      required: true,
    },
    timeout: {
      type: Number,
      required: false,
      default: 5000 * 2,
    },
  },
  computed: {
    showSnack: {
      get() {
        return this.show;
      },
      set() {
        this.$emit('close');
      },
    },
  },
};
</script>
