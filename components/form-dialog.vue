<template>
  <v-row justify="center">
    <v-dialog v-model="dialogController" persistent max-width="60%">
      <v-layout align-center justify-center>
        <slot></slot>
      </v-layout>
    </v-dialog>
  </v-row>
</template>
<script>
export default {
  props: ["dialog"],
  data() {
    return { dialogController: false };
  },
  watch: {
    dialog: {
      immediate: true,
      handler() {
        this.dialogController = this.dialog;
      },
    },
  },
};
</script>

<style>
</style>
