<template>
  <div>
    <div v-if="!isLoggedIn && isShowLogin">
      <form action="" @submit.prevent>
        <div class="mb-6">
          <FormTextField
            v-model="login.username"
            placeholder="Username or email address *"
          />
        </div>
        <div class="mb-6">
          <FormTextField
            v-model="login.password"
            placeholder="Password"
            :type="'password'"
          />
        </div>
        <div class="flex items-center justify-between mb-6">
          <div class="text-second text-sm">
            <input type="checkbox" name="" id="" />
            Remember me
          </div>
          <button class="text-head underline text-sm">Lost password?</button>
        </div>
        <button
          class="w-full px-8 py-4 bg-head text-center text-white text-sm mb-6"
          @click="customerLogin"
        >
          LOGIN
        </button>
        <p class="text-sm text-second text-center">
          No account yet?
          <span
            class="text-head underline cursor-pointer"
            @click="isShowLogin = false"
            >Create Account</span
          >
        </p>
      </form>
    </div>
    <!-- Create Account Form -->
    <div v-if="!isLoggedIn && !isShowLogin">
      <ValidationObserver v-slot="{ invalid }">
        <form @submit.prevent>
          <div class="mb-6">
            <FormTextField
              v-model="customerRegister.username"
              placeholder="Username"
              rules="required|string"
            />
          </div>
          <div class="mb-6">
            <FormTextField
              v-model="customerRegister.password"
              placeholder="Password"
              :type="'password'"
              rules="required|min:4"
              vid="password"
            />
          </div>
          <div class="mb-6">
            <FormTextField
              v-model="confirmPassword"
              placeholder="Confirm Password"
              :type="'password'"
              rules="required|confirmed:password"
            />
          </div>
          <div class="mb-6">
            <FormTextField
              v-model="customerRegister.first_name"
              placeholder="First Name"
              rules="required|string"
            />
          </div>
          <div class="mb-6">
            <FormTextField
              v-model="customerRegister.last_name"
              placeholder="Last Name"
              rules="required|string"
            />
          </div>
          <div class="mb-6">
            <FormTextField
              v-model="customerRegister.email"
              placeholder="Email address *"
              type="email"
              required
              rules="required|email"
            />
          </div>

          <p class="text-second text-sm text-justify mb-6">
            Your personal data will be used to support your experience
            throughout this website, to manage access to your account, and for
            other purposes described in our privacy policy.
          </p>
          <div class="flex">
            <ReusableLoaderButton
              label="REGISTER"
              @click="registerCustomer"
              width="w-full"
              :disabled="invalid"
            />
          </div>
        </form>
      </ValidationObserver>
    </div>
    <div v-if="isLoggedIn">
      <ReusableLoaderButton label="LOG OUT" @click="logOut" width="w-full" />
    </div>
  </div>
</template>

<script>
import axios from "~/plugins/axios";
import { mapMutations, mapState } from "vuex";
export default {
  name: "LoginRegisterForm",
  data() {
    return {
      isShowLogin: true,
      customerRegister: {},
      login: {
        username: "pachalam",
        password: "1234567",
      },
      confirmPassword: null,
      loading: false,
    };
  },
  computed: {
    ...mapState(["isLoggedIn"]),
  },
  methods: {
    // ...mapMutations("user", ["setAccess"]),
    ...mapMutations(["setLoggedIn","setAccess","setUser"]),
    async customerLogin() {
      try {
        const { data } = await this.$api.post("/account/token/user/login/", {
          ...this.login,
        });

        const tokensString = data.tokens;
        const validJsonString = tokensString.replace(/'/g, '"');
        const tokenObj = JSON.parse(validJsonString);
        const { refresh, access } = tokenObj;

        localStorage.setItem("refresh", refresh);
        this.setAccess(access);
        this.setLoggedIn(true);
        this.$alert.show({
          title: "Logged in Successfully",
          description: "You have successfully logged in!",
        });
        // this.$router.push("/");
        this.$emit("loggedIn");
      } catch (error) {
        console.log("er", error);
        this.$alert.show({
          isError: true,
          title: "Invalid Login",
          description: "Please enter valid credentials",
        });
      }
    },
    async registerCustomer() {
      try {
        this.loading = true;

        const res = this.$api.post(
          "/account/user/sign-up/",
          this.customerRegister
        );

        // const res = await this.$axios.$post(
        //   "/api/customers",
        //   this.customerRegister
        // );

        this.$alert.show({
          title: "Yasss, account created.",
          description:
            "Your account have been created successfully. Please login with your email and password!",
        });
      } catch (error) {
      } finally {
        this.loading = false;
        this.isShowLogin = true;
      }
    },
    async logOut() {
      try {
        await this.$api.post("/account/user/logout/");
        localStorage.removeItem("refresh");
        this.setLoggedIn(false);
        this.setAccess("");
        this.setUser({})
        this.$emit("logout")
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<style></style>
