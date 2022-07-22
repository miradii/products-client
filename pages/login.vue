<script>
import { required, min, email, max, confirmed } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";

import axios from "axios";
extend("max", {
  ...max,
  message: "{_field_} may not be more than {length} characters",
});

extend("min", {
  ...min,
  message: "{_field_} must be at least {length} characters",
});

extend("email", {
  ...email,
  message: "{_field_} must be valid E-mail address",
});

extend("required", {
  ...required,
  message: "{_field_} can not be empty",
});

export default {
  layout: "main",
  components: { ValidationObserver, ValidationProvider },
  data() {
    return {
      email: "",
      password: "",
    };
  },

  methods: {
    validate() {
      this.$refs.observer.validate();
    },
    async handleLogin() {
      try {
        const info = { email: this.email, password: this.password };
        const res = await this.$auth.loginWith("local", {
          data: info,
        });
        this.$store.commit("snackbar/showMessage", {
          content: "You are now logged in",
          color: "success",
        });
      } catch (e) {
        const errors = e.response.data.message;
        if (typeof errors == "string") {
          console.log(errors);
        } else if (errors !== undefined) {
          errors.forEach((error) => {
            if (error.includes("email")) {
              this.$refs.email.setErrors(error);
            } else if (error.includes("password")) {
              this.$refs.password.setErrors(error);
            }
          });
        }
      }
    },
  },
};
</script>


<template>
  <v-layout align-center justify-center>
    <v-card class="pa-4" width="60%">
      <ValidationObserver ref="observer" v-slot="{ invalid }">
        <v-form ref="form">
          <validation-provider
            ref="email"
            v-slot="{ errors }"
            name="E-Mail"
            vid="email"
            rules="required|email"
          >
            <v-text-field
              class="ma-2"
              v-model="email"
              :error-messages="errors"
              @change="validate"
              name="email"
              label="Email"
              type="text"
              required
              outlined
            ></v-text-field>
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            name="password"
            ref="password"
            vid="password"
            rules="required|max:80|min:6"
          >
            <v-text-field
              v-model="password"
              :error-messages="errors"
              @change="validate"
              id="password"
              required
              class="ma-2"
              name="password"
              label="Password"
              type="password"
              outlined
            ></v-text-field>
          </validation-provider>
        </v-form>
        <v-card-actions>
          <v-btn
            :disabled="invalid"
            @click.prevent="handleLogin"
            large
            color="green accent-2"
            light
          >Login</v-btn>
        </v-card-actions>
      </ValidationObserver>
    </v-card>
  </v-layout>
</template>


<style>
</style>
