<template lang="pug">
login-layout
  template(slot="title") Forgot Password
  template(slot="body")
    v-form(v-model="validForm" ref="form" @submit.prevent="requestReset()" lazy-validation)
      p.body-1 Please enter your username and request a password reset code.
      p.body-1 The code will be sent to the email associated with your account. Please contact an admin if you experience any issues.
      username-field(v-model="username")

  template(slot="footer")
    v-spacer
    router-link(:to="{name:'VerifyAccount'}") I already have a password reset code

  template(slot="actions")
    login-button(@click="$router.push({name: 'LoginUsername'})") Back
    login-button(primary @click="requestReset()") Request Reset

</template>

<script>
import LoginLayout from "./../layouts/Login";
import PasswordField from "@/components/PasswordField";
import UsernameField from "@/components/UsernameField";
import LoginButton from "@/components/Button";

export default {
  name: "ForgotPassword",
  components: {
    LoginButton,
    LoginLayout,
    PasswordField,
    UsernameField
  },
  data() {
    return {
      validForm: null
    };
  },
  computed: {
    username: {
      get() {
        return this.$store.getters.username;
      },
      set(value) {
        return this.$store.dispatch("updateUsername", value);
      }
    }
  },
  methods: {
    requestReset() {
      this.errors = [];
      if (!this.$refs.form.validate()) {
        return false;
      }
      this.$store
        .dispatch("apiCall", { callName: "verifyUsername" })
        .then(response => {
          if (response.exists && response.setup) {
            this.$store.dispatch("apiCall", {
              callName: "requestToken",
              username: this.username
            });
            this.$store.dispatch(
              "message",
              "A password-reset code has been sent to your email."
            );
            this.$router.push({ name: "VerifyAccount" });
          } else if (response.exists && !response.setup) {
            this.$router.push({ name: "UnverifiedAccount" });
          } else {
            this.errors.push(
              `An account with the username "${this.username}" does not exist.`
            );
          }
        });
    }
  }
};
</script>
