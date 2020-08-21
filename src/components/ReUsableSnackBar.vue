<template>
  <v-snackbar
    v-model="showSnack"
    :timeout="5000 * 2"
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
