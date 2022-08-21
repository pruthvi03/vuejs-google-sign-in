<template>
  <div>
    <v-btn class="sign-in-btn" color="white" tile @click="signInWithGoogle">
      <v-img
        src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/53/Google_%22G%22_Logo.svg/512px-Google_%22G%22_Logo.svg.png"
        max-width="20px"
        class="mr-2"
      >
      </v-img>
      Register with google
    </v-btn>
  </div>
</template>
<script>
import { mapGetters } from "vuex";
import firebase from "firebase/compat/app";
import "firebase/compat/firestore";
import { getAuth, GoogleAuthProvider, signInWithPopup } from "firebase/auth";
export default {
  data() {
    return {
      db: null,
    };
  },
  mounted() {},
  computed: {
    ...mapGetters({
      user: "user",
    }),
  },
  methods: {
    async signInWithGoogle() {
      try {
        const provider = new GoogleAuthProvider();
        const result = await signInWithPopup(getAuth(), provider);
        this.$store.commit("SET_USER", {
          uid: result.user.uid,
          email: result.user.email,
          displayName: result.user.displayName,
        });
        this.$store.commit("SET_LOGGED_IN", true);
        await this.createUser();
      } catch (error) {
        console.log(error);
      }
    },
    async createUser() {
      const userRef = firebase
        .firestore()
        .collection("users")
        .doc(this.user.data.uid);
      const userDoc = await userRef.get();
      if (!userDoc.exist)
        await userRef.set(
          {
            firstName: null,
            lastName: null,
            phoneNumber: null,
            dateOfBirth: null,
            age: null,
            gender: null,
            info: null,
            country: null,
            state: null,
            zipcode: null,
            ...this.$store.state.user.data,
          },
          { merge: true }
        );
    },
  },
};
</script>

<style lang="scss" scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600&display=swap");

.sign-in-btn {
  font-family: "poppins";
  font-weight: 500;
  background: #fff;
  box-shadow: none;
  width: 440px;
  height: 48px;
  padding: 28px 300px !important;
  border: 1px solid #205eff !important;
  color: #205eff !important;
}
</style>
