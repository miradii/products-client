 <template>
  <v-app>
    <v-app-bar flat app>
      <v-container class="py-0 fill-height">
        <v-btn class="mx-2" to="/" router text>Welcome</v-btn>
        <v-btn class="mx-2" v-if="$auth.loggedIn" to="/categories" router text>Categories</v-btn>
        <v-btn class="mx-2" v-if="$auth.loggedIn" to="/products" router text>Products</v-btn>
        <v-spacer></v-spacer>
        <div v-if="!$auth.loggedIn">
          <v-btn
            v-for="item in items"
            :to="item.to"
            :key="item.title"
            class="mx-2"
            router
            text
          >{{item.title }}</v-btn>
        </div>
        <div v-else>
          <v-btn class="mx-2" @click="logout" text>Log Out</v-btn>
        </div>
      </v-container>
    </v-app-bar>
    <v-main>
      <v-container>
        <snack-bar></snack-bar>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer absolute app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import SnackBar from "../components/snack-bar.vue";
export default {
  name: "main",
  components: { SnackBar },

  data() {
    return {
      drawer: false,
      items: [
        {
          title: "Login",
          to: "/login",
          auth: false,
        },
        {
          title: "Sign Up",
          to: "/signup",
        },
      ],

      title: "Product Viewer",
    };
  },
  methods: {
    async logout() {
      await this.$auth.logout();
      this.$store.commit("snackbar/showMessage", {
        content: "You logged out. please come back later",
        color: "success",
      });
    },
  },
};
</script>

<style>
</style>
