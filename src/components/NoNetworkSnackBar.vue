<template>
  <v-snackbar
    v-model="network"
    :timeout="5000 * 2"
  >
    No Intrenet connection.
  </v-snackbar>
</template>

<script>
export default {
  name: 'NoNetworkSnackBar',
  props: {
    noInternet: {
      type: Boolean,
      required: true,
    },
  },
  computed: {
    network: {
      get() {
        return this.noInternet;
      },
      set() {
        this.$emit('close');
      },
    },
  },
};
</script>
