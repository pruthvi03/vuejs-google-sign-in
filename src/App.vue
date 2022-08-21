<template>
  <v-app>
    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script>
import { getAuth } from "@firebase/auth";
export default {
  name: "App",

  created() {
    const auth = getAuth();
    auth.onAuthStateChanged((user) => {
      this.$store.dispatch("fetchUser", user);
    });
  },

  data: () => ({
    //
  }),
};
</script>
