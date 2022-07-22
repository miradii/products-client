<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card class="pa-4 justify-center">
        <h1>
          Welcome
          <span v-if="$auth.loggedIn">{{$auth.user.name}}</span>
        </h1>
        <h2>This is a sample app for testing the products api</h2>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  layout: "main",
  name: "IndexPage",
  mounted() {
    console.log(this.$auth.loggedIn);
  },
};
</script>
