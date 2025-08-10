<template>
  <div id="app">
    <!-- Top-right auth nav -->
    <NavigationAuth
      :isLoggedIn="isLoggedIn"
      @setActivePage="setActivePage"
      @logOut="logOut"
    />

    <!-- Main nav (only when logged in) -->
    <NavigationComponent
      v-if="isLoggedIn"
      @setActivePage="setActivePage"
    />

    <!-- Pages -->
    <Page_Swipe v-if="activePage === 'swipe'"
      :user="loggedInUser"
    />

    <Page_Profile v-if="activePage === 'profile'"
      :user="loggedInUser"
    />

    <Page_Messages v-if="activePage === 'messages'"
      :user="loggedInUser"
    />

    <!-- Auth -->
    <Page_login v-if="activePage === 'login'"
      @setActivePage="setActivePage"
      @loginSuccess="handleLogin"
    />

    <Page_signup v-if="activePage === 'signup'"
      @setActivePage="setActivePage"
      @loginSuccess="handleLogin"
    />

    <Page_logout v-if="activePage === 'logout'"
      @setActivePage="setActivePage"
      @logOut="logOut"
    />
  </div>
</template>

<script>
import NavigationAuth from "./components/NavigationAuth.vue";
import NavigationComponent from "./components/NavigationComponent.vue";

import Page_login from "./pages/Page_login.vue";
import Page_signup from "./pages/Page_signup.vue";
import Page_logout from "./pages/Page_logout.vue";
import Page_Swipe from "./pages/Page_Swipe.vue";
import Page_Profile from "./pages/Page_Profile.vue";
import Page_Messages from "./pages/Page_Messages.vue";

export default {
  name: "App",
  components: {
    NavigationAuth,
    NavigationComponent,
    Page_login,
    Page_signup,
    Page_logout,
    Page_Swipe,
    Page_Profile,
    Page_Messages,
  },
  data() {
    return {
      activePage: "login",
      isLoggedIn: false,
      loggedInUser: null, // expect { user_id, username }
    };
  },
  methods: {
    setActivePage(page) {
      this.activePage = page;
    },
    logOut() {
      this.isLoggedIn = false;
      this.loggedInUser = null;
      this.activePage = "login";
    },
    handleLogin(user) {
      this.loggedInUser = user;
      this.isLoggedIn = true;
      this.activePage = "swipe";
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
