<template>
  <v-snackbar v-model="show" height="40px" :color="color" :timeout="timeout">
    {{ message }}
    <template v-slot:action="{ attrs }">
      <v-btn color="green" v-bind="attrs" @click="show = false">Close</v-btn>
    </template>
  </v-snackbar>
</template>

<script>
export default {
  data() {
    return {
      message: "",
      color: "",
      show: false,
      timeout: 8000,
    };
  },
  created() {
    this.$store.subscribe((mutation, state) => {
      if (mutation.type === "snackbar/showMessage") {
        this.message = state.snackbar.content;
        this.color = state.snackbar.color;
        this.show = true;
      }
    });
  },
};
</script>
