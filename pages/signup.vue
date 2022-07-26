
<script>
import { required, min, email, max, confirmed } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";

import axios from "axios";

extend("email", {
  ...email,
  message: "{_field_} must be valid E-mail address",
});

extend("required", {
  ...required,
  message: "{_field_} can not be empty",
});

extend("max", {
  ...max,
  message: "{_field_} may not be more than {length} characters",
});

extend("min", {
  ...min,
  message: "{_field_} must be at least {length} characters",
});
extend("confirmed", {
  ...confirmed,
  message: "{_field_}s do not match",
});

export default {
  layout: "main",
  components: { ValidationObserver, ValidationProvider },
  data() {
    return {
      name: "",
      email: "",
      password: "",
      confirm: "",
    };
  },

  methods: {
    async handleSignup() {
      try {
        const user = {
          name: this.name,
          email: this.email,
          password: this.password,
          confirmPassword: this.confirm,
        };

        const res = await this.$axios.post("/api/signup", user);
        this.$store.commit("snackbar/showMessage", {
          content: "You signed up, you can now log in",
          color: "success",
        });
        this.$router.push("/login");
      } catch (e) {
        const errors = e.response.data.message;
        console.log(errors);
        errors.forEach((error) => {
          if (error.includes("email")) {
            this.$refs.email.setErrors(error);
          } else if (error.includes("password")) {
            this.$refs.password.setErrors(error);
          } else if (error.includes("name")) {
            this.$refs.name.setErrors(error);
          } else if (error.includes("confirm")) {
            this.$refs.name.setErrors(error);
          }
        });
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
              name="email"
              label="Email"
              type="text"
              required
              outlined
            ></v-text-field>
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            name="Name"
            vid="name"
            ref="name"
            rules="required|max:80|min:5"
          >
            <v-text-field
              class="ma-2"
              v-model="name"
              required
              :error-messages="errors"
              name="fullName"
              label="Full Name"
              type="text"
              outlined
            ></v-text-field>
          </validation-provider>

          <validation-provider
            v-slot="{ errors }"
            name="password"
            ref="password"
            vid="password"
            rules="required|max:80|min:6|confirmed:confirm"
          >
            <v-text-field
              v-model="password"
              :error-messages="errors"
              id="password"
              required
              class="ma-2"
              name="password"
              label="Password"
              type="password"
              outlined
            ></v-text-field>
          </validation-provider>
          <ValidationProvider v-slot="{ errors }" vid="confirm">
            <v-text-field
              id="confirm"
              v-model="confirm"
              :error-messages="errors"
              required
              class="ma-2"
              ref="confirm"
              name="confirm"
              label="Confirm Password"
              type="password"
              outlined
            ></v-text-field>
          </ValidationProvider>
        </v-form>
        <v-card-actions>
          <v-btn
            :disabled="invalid"
            @click.prevent="handleSignup"
            large
            color="green accent-2"
            light
          >Sign Up</v-btn>
        </v-card-actions>
      </ValidationObserver>
    </v-card>
  </v-layout>
</template>
<style>
</style>
