<template>
  <v-layout align-center justify-center>
    <v-card class="pa-4" width="60%">
      <v-form>
        <v-text-field class="ma-2" name="phoneNumber" label="Email" type="text" outlined></v-text-field>
        <v-text-field class="ma-2" name="firstName" label="Full Name" type="text" outlined></v-text-field>
        <v-text-field
          id="password"
          class="ma-2"
          name="password"
          label="Password"
          type="password"
          outlined
        ></v-text-field>
        <v-text-field
          id="confirm"
          class="ma-2"
          name="confirm"
          label="Confirm Password"
          type="password"
          outlined
        ></v-text-field>
      </v-form>

      <v-card-actions>
        <v-btn large color="green accent-2" light>Sign Up</v-btn>
      </v-card-actions>
    </v-card>
  </v-layout>
</template>

<script>
export default {
  layout: "main",
};
</script>

<style>
</style>
