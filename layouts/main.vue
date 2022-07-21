 <template>
  <v-app>
    <v-app-bar flat app>
      <v-container class="py-0 fill-height">
        <v-btn
          v-for="item in items"
          :to="item.to"
          :key="item.title"
          class="mx-2"
          router
          text
        >{{item.title }}</v-btn>

        <v-spacer></v-spacer>
      </v-container>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer :absolute="!fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  name: "main",

  data() {
    return {
      drawer: false,
      items: [
        {
          icon: "mdi-apps",
          title: "Welcome",
          to: "/",
        },
        {
          icon: "mdi-chart-bubble",
          title: "Signup",
          to: "/signup",
        },
      ],
      title: "Product Viewer",
    };
  },
};
</script>

<style>
</style>
