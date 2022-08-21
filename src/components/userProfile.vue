<template>
  <div v-if="!$store.state.user.isLoggedIn" class="google-login">
    <google-sign-in />
  </div>
  <div v-else class="user-container">
    <div class="user-photo mt-15">
      <v-avatar style="width: 300px; height: 300px">
        <img
          :src="image || 'https://cdn.vuetifyjs.com/images/john.jpg'"
          alt="profile_pic"
        />
      </v-avatar>
      <v-spacer />
    </div>
    <v-form ref="userForm" v-model="isFormValid" @submit="storeData">
      <div class="user-info">
        <div class="d-flex flex-row align-center title">
          <div class="text-uppercase mr-10 underline-text">
            General Informantion
          </div>
          <div class="m-10 body-1" style="color: red; font-weight: 500">
            Log out<v-btn icon color="red" @click="signOutFromGoogle">
              <v-icon small>mdi-logout</v-icon>
            </v-btn>
          </div>
        </div>
        <div class="user-form mt-10 d-flex flex-wrap">
          <div class="form-item">
            <small>First name *</small>
            <v-text-field
              solo
              v-model="firstName"
              name="firstName"
              :rules="rules"
            />
          </div>
          <div class="form-item">
            <small>Last name *</small>
            <v-text-field
              solo
              v-model="lastName"
              name="lastName"
              :rules="rules"
            />
          </div>
          <div class="form-item">
            <small>Email *</small>
            <v-text-field
              solo
              prepend-inner-icon="mdi-email"
              name="email"
              v-model="email"
              readonly
            />
          </div>
          <div class="form-item">
            <small>Phone number *</small>
            <v-text-field
              solo
              prepend-inner-icon="mdi-phone"
              name="phone"
              v-model="phoneNumber"
              :rules="rules"
            />
          </div>
          <div class="d-flex age-component">
            <div class="mb-2">
              <small>Date of birth *</small><br />
              <date-picker
                v-model="dateOfBirth"
                class="data-of-birth"
                type="date"
                format="YYYY-MM-DD"
                style="color: #3b3b3b; font-weight: 500"
              >
              </date-picker>
              <!-- <small v-if="birthDateRule" style="color: red">{{
                birthDateRule
              }}</small> -->
            </div>
            <div class="d-flex align-self-end mb-2" style="min-width: 350px">
              <small
                class="mt-2 mx-4 d-flex"
                style="color: #3b3b3b; font-weight: 500"
                >Age <span class="ml-2 body1">{{ findAge }} y</span></small
              >
              <v-chip
                outlined
                pill
                color="darkgray"
                class="ml-5"
                style="color: #3b3b3b; font-weight: 500"
                >{{ ageGroup }}</v-chip
              >
            </div>
          </div>

          <div class="mx-3 mt-3" style="width: 100%">
            <small>Select your gender</small>
            <v-radio-group row v-model="gender">
              <v-radio
                v-for="n of genderOptions"
                :key="n"
                :label="n"
                :value="n"
                style="font-weight: 500px"
              ></v-radio>
            </v-radio-group>
          </div>
          <div class="about-text-area" style="width: 100%">
            <small>About me</small>
            <v-textarea solo label="I am ..." auto-grew v-model="info">{{
              info
            }}</v-textarea>
          </div>

          <div class="d-flex justify-space-between address" style="width: 100%">
            <div class="address-item">
              <small>Country * </small>
              <v-select
                v-model="country"
                :items="countryOptions"
                :rules="rules"
                label="Country"
                solo
              ></v-select>
            </div>

            <div class="address-item">
              <small>State </small>
              <v-select
                v-model="state"
                :items="stateOptions"
                label="State"
                solo
              ></v-select>
            </div>

            <div class="address-item">
              <small>ZIP code *</small>
              <v-text-field v-model="zipcode" :rules="rules" solo />
            </div>
          </div>

          <v-btn
            :disabled="!isFormValid"
            color="#205EFF"
            style="width: 200px"
            @click="storeData"
            :loading="loading"
            ><span style="color: #fff">Save</span></v-btn
          >
        </div>
      </div>
    </v-form>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import firebase from "firebase/compat/app";
import { getAuth } from "@firebase/auth";
import googleSignIn from "./googleSignIn.vue";
import DatePicker from "vue2-datepicker";
import "firebase/compat/firestore";
import "vue2-datepicker/index.css";

export default {
  components: { googleSignIn, DatePicker },
  name: "UserProfile",

  data: () => ({
    isFormValid: false,
    firstName: "",
    lastName: "",
    file: null,
    email: "",
    phoneNumber: "",
    dateOfBirth: "",
    age: 30,
    gender: "",
    genderOptions: ["Male", "Female", "Others"],
    info: "",
    country: "",
    state: "",
    zipcode: "",
    countryOptions: ["United States", "India", "Germany"],
    stateOptions: ["Texas", "Gujarat"],
    image: null,
    rules: [(v) => !!v || "This field is required"],
    loading: false,
  }),
  computed: {
    ...mapGetters({
      user: "user",
    }),
    ageGroup() {
      return this.age >= 18 ? "Adult 18+" : "minor";
    },
    findAge() {
      if (this.dateOfBirth) {
        const today = new Date();
        const birthDate = this.dateOfBirth;
        if (birthDate >= today) {
          this.age = 0;
          return 0;
        }
        let age = today.getFullYear() - birthDate.getFullYear();
        const m = today.getMonth() - birthDate.getMonth();
        if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
          age--;
        }
        this.age = age;
        return age;
      }
      return this.age;
    },
    birthDateRule(value) {
      console.log(value, this.findAge);
      if (this.findAge <= 0) {
        return "Age should be greater than 0";
      } else {
        return true;
      }
    },
  },
  async mounted() {
    setTimeout(() => {
      this.fetchUserData();
    }, 3000);
  },
  methods: {
    async fetchUserData() {
      try {
        this.loading = true;
        const uid = this.$store.state.user.data.uid;
        this.email = this.$store.state.user.data.email;
        const userDoc = await firebase
          .firestore()
          .collection("users")
          .doc(uid)
          .get();
        if (userDoc.exists) {
          const userData = userDoc.data();
          console.log(userData);
          this.firstName = userData.firstName;
          this.lastName = userData.lastName;
          this.email = this.$store.state.user.data.email;
          this.phoneNumber = userData.phoneNumber;
          this.dateOfBirth =
            userData.dateOfBirth || userData.dateOfBirth.toDate();
          this.age = userData.age;
          this.gender = userData.gender;
          this.info = userData.info;
          this.country = userData.country;
          this.state = userData.state;
          this.zipcode = userData.zipcode;
        }
      } catch (error) {
        console.log(error);
      } finally {
        this.loading = false;
      }
    },
    async storeData() {
      try {
        this.loading = true;
        await firebase
          .firestore()
          .collection("users")
          .doc(this.user.data.uid)
          .update({
            firstName: this.firstName,
            lastName: this.lastName,
            email: this.email,
            phoneNumber: this.phoneNumber,
            dateOfBirth: this.dateOfBirth,
            age: this.age,
            gender: this.gender,
            info: this.info,
            country: this.country,
            state: this.state,
            zipcode: this.zipcode,
          });
        this.fetchUserData();
      } catch (error) {
        console.log(error);
      } finally {
        this.loading = false;
      }
    },
    signOutFromGoogle() {
      this.$refs.userForm.reset();
      this.dateOfBirth = null;
      const auth = getAuth();
      auth
        .signOut()
        .then(() => console.log("Sign out user"))
        .catch((err) => console.log(err));
    },
  },
};
</script>
<style>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600&display=swap");

.google-login {
  margin: 0;
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.user-container {
  font-family: "Poppins", sans-serif;
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 10px 100px;
  padding: 100px;
  width: 100%;
}

.age {
  height: 50px;
  width: 100px;
  border: none !important;
  background: transparent !important;
  box-shadow: none !important;
}

.form-item .v-input__slot,
.about-text-area .v-input__slot,
.address .v-input__slot {
  background: transparent !important;
  box-shadow: none !important;
  border: 1px solid #e4e4e4;
  color: #3b3b3b;
  font-weight: 500;
}

.user-form {
  max-width: 700px;
}

.form-item {
  width: 320px;
  margin: 5px 25px 10px 0px;
}

.underline-text {
  font-family: sans-serif;
  text-decoration: none;
  color: #205eff;
  font-weight: 600;
  letter-spacing: 2px;
  font-size: 16px;
  border-bottom: 2px solid #205eff;
  padding-bottom: 2px;
}

.address-item {
  width: 200px;
}
</style>
