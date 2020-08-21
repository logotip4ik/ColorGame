<template>
  <v-dialog
    persistent
    v-model="successModel"
    max-width="500px">
    <v-card
      :loading="loading"
      dark>
      <v-card-title class="headline">
        Hoorey you guess color 🎉
      </v-card-title>
      <v-card-text>
        Congratulations this was {{color}} with name:
         <pre class="font-weight-bold" style="font-size: 20px">{{name}}</pre>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: 'SuccessOverlay',
  props: {
    success: {
      type: Boolean,
      required: true,
    },
    color: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    loading: {
      type: Boolean,
      required: true,
    },
  },
  computed: {
    successModel: {
      get() {
        return this.success;
      },
      set() {
        this.$emit('close');
      },
    },
  },
};
</script>
