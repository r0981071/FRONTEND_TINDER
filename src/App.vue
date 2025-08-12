<script>
import NavigationComponent from './components/NavigationComponent.vue';
import NavigationAuth from './components/NavigationAuth.vue';

import Page_Swipe from './pages/Page_Swipe.vue';
import Page_Profile from './pages/Page_Profile.vue';
import Page_Matches from './pages/Page_Matches.vue';
import Page_Messages from './pages/Page_Messages.vue';

import Page_login from './pages/Page_login.vue';
import Page_signup from './pages/Page_signup.vue';
import Page_logout from './pages/Page_logout.vue';

export default {
  name: 'App',
  data() {
    return {
      activePage: 'login',      // <-- start at login
      isLoggedIn: false,
      loggedInUser: null,
      chatWithUserId: null,
    };
  },
  components: {
    NavigationComponent,
    NavigationAuth,
    Page_Swipe,
    Page_Profile,
    Page_Matches,
    Page_Messages,
    Page_login,
    Page_signup,
    Page_logout,
  },
  methods: {
    setActivePage(page) { this.activePage = page; },
    logOut() {
      this.isLoggedIn = false;
      this.loggedInUser = null;
      this.chatWithUserId = null;
      this.activePage = 'login'; // <-- go back to login
    },
    handleLogin(user) {
      this.loggedInUser = user;
      this.isLoggedIn = true;
      this.activePage = 'swipe'; // <-- go to swipe after login
    },
    openChat(otherId) {
      this.chatWithUserId = Number(otherId);
      this.activePage = 'messages';
    },
  }
};
</script>

<template>
  <div id="app">
    <NavigationAuth
      @setActivePage="setActivePage"
      @logOut="logOut"
      :isLoggedIn="isLoggedIn"
    />

    <NavigationComponent
      v-if="isLoggedIn"
      @setActivePage="setActivePage"
    />

    <!-- Auth pages (only when NOT logged in) -->
    <Page_login
      v-if="!isLoggedIn && activePage === 'login'"
      @setActivePage="setActivePage"
      @loginSuccess="handleLogin"
    />
    <Page_signup
      v-if="!isLoggedIn && activePage === 'signup'"
      @setActivePage="setActivePage"
      @loginSuccess="handleLogin"
    />

    <!-- Logout (only when logged in) -->
    <Page_logout
      v-if="isLoggedIn && activePage === 'logout'"
      @setActivePage="setActivePage"
      @logOut="logOut"
    />

    <!-- App pages (only when logged in) -->
    <Page_Swipe v-if="isLoggedIn && activePage === 'swipe'" :user="loggedInUser" />
    <Page_Profile v-if="isLoggedIn && activePage === 'profile'" :user="loggedInUser" />
    <Page_Matches
      v-if="isLoggedIn && activePage === 'matches'"
      :user="loggedInUser"
      @openChat="openChat"
      @setActivePage="setActivePage"
    />
    <Page_Messages
      v-if="isLoggedIn && activePage === 'messages'"
      :user="loggedInUser"
      :initialOtherId="chatWithUserId"
    />
  </div>
</template>
