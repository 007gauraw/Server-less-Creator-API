<template>
  <div>
    <div id="logo">
      <img :src="'./logo.png'" />
    </div>
    <div style="margin-bottom: 1em;">
      <button v-if="!user" @click="login">Login</button>
      <button v-if="user" @click="logout">Logout</button>
      <span v-if="user">Logged in as: {{ user.profile.email }}</span>
    </div>
    <h2>Hi, I Am SAM</h2>
    <hr />  
    <h3>Create User</h3>
    <CreateItem class="form-section"/>
    <hr />  
    <h3>Get User By ID</h3>
    <GetItemById class="form-section"/>
    <hr />    
    <h3>Get All Users</h3>
    <GetItems class="form-section"/>
  </div>
</template>

<script>
import CreateItem from './components/CreateItem'
import GetItems from './components/GetItems'
import GetItemById from './components/GetItemById'
import { UserManager, WebStorageStateStore } from 'oidc-client-ts';

const cognitoAuthConfig = {
  authority: "https://cognito-idp.ap-south-1.amazonaws.com/ap-south-1_ZnZ0C8oXd",
  client_id: "1v16trkjiq6nbbg8kh9cb5jnpq",
  redirect_uri: window.location.origin,
  response_type: "code",
  scope: "aws.cognito.signin.user.admin email openid phone",
  userStore: new WebStorageStateStore({ store: window.localStorage })
};

const userManager = new UserManager(cognitoAuthConfig);

export default {
  name: 'App',
  components: {
    GetItems,
    CreateItem,
    GetItemById,
  },
  data() {
    return {
      user: null,
    };
  },
  created() {
    userManager.getUser().then(user => {
      if (!user || user.expired) {
        // Check for redirect callback
        if (window.location.search.includes('code=')) {
          userManager.signinRedirectCallback().then(user => {
            this.user = user;
            window.history.replaceState({}, document.title, window.location.pathname);
          });
        }
      } else {
        this.user = user;
      }
    });

    // Intercept fetch to add Authorization header
    const origFetch = window.fetch;
    window.fetch = async (input, init = {}) => {
      const user = await userManager.getUser();
      if (user && user.access_token) {
        init.headers = init.headers || {};
        if (init.headers instanceof Headers) {
          init.headers.set('Authorization', `Bearer ${user.access_token}`);
        } else {
          init.headers['Authorization'] = `Bearer ${user.access_token}`;
        }
      }
      return origFetch(input, init);
    };
  },
  methods: {
    login() {
      userManager.signinRedirect();
    },
    logout() {
      userManager.signoutRedirect();
      this.user = null;
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#logo {
    display: flex;
    width: 15%;
    flex-direction: column;
    justify-content: center;
    margin: auto;
}

.form-section {
  display: block;
  margin: 2%;
  justify-content: left;
}

.error {
  color: red;
}

div {
  margin: 1%;
}
input {
  margin: 1%;
}
label {
  margin: 1%;
}

</style>
