<script>
import NavigationComponent from './components/NavigationComponent.vue';
import NavigationAuth from './components/NavigationAuth.vue';

import Page_Swipe from './pages/Page_Swipe.vue';
import Page_Profile from './pages/Page_Profile.vue';
import Page_Matches from './pages/Page_Matches.vue';
import Page_Messages from './pages/Page_Messages.vue';
import Page_Admin from './pages/Page_Admin.vue'; // ⬅ NEW

import Page_login from './pages/Page_login.vue';
import Page_signup from './pages/Page_signup.vue';
import Page_logout from './pages/Page_logout.vue';

export default {
  name: 'App',
  data() {
    return {
      activePage: 'login',
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
    Page_Admin, // ⬅ NEW
    Page_login,
    Page_signup,
    Page_logout,
  },
  computed: {
    isAdmin() {
      return this.loggedInUser && this.loggedInUser.username === 'admin';
    }
  },
  methods: {
    setActivePage(page) { this.activePage = page; },
    logOut() {
      this.isLoggedIn = false;
      this.loggedInUser = null;
      this.chatWithUserId = null;
      this.activePage = 'login';
    },
    handleLogin(user) {
      this.loggedInUser = user;
      this.isLoggedIn = true;
      // ⬇ admin goes to admin; normal user to swipe
      this.activePage = (user.username === 'admin') ? 'admin' : 'swipe';
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

    <!-- Hide the normal nav for admins -->
    <NavigationComponent
      v-if="isLoggedIn && !isAdmin"
      @setActivePage="setActivePage"
    />

    <!-- Auth pages -->
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
    <Page_logout
      v-if="isLoggedIn && activePage === 'logout'"
      @setActivePage="setActivePage"
      @logOut="logOut"
    />

    <!-- Admin page (only when admin) -->
    <Page_Admin v-if="isLoggedIn && isAdmin && activePage === 'admin'" />

    <!-- Normal user pages -->
    <Page_Swipe   v-if="isLoggedIn && !isAdmin && activePage === 'swipe'"    :user="loggedInUser" />
    <Page_Profile v-if="isLoggedIn && !isAdmin && activePage === 'profile'"  :user="loggedInUser" />
    <Page_Matches v-if="isLoggedIn && !isAdmin && activePage === 'matches'"  :user="loggedInUser" @openChat="openChat" @setActivePage="setActivePage" />
    <Page_Messages v-if="isLoggedIn && !isAdmin && activePage === 'messages'" :user="loggedInUser" :initialOtherId="chatWithUserId" />
  </div>
</template>
