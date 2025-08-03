<template>
  <div>    
    <form @submit.prevent="GetItemById">
      <div>
        <label for="userId">User ID</label>
        <input type="text" id="userId" v-model="formData.userId" />
      </div>
      <div>
        <button @click="getItemsById">Get User</button>
      </div>
    </form>
    <h3 v-if="user.id">{{ user.id }} . {{ user.name }}</h3>      
    <h3 class="error" v-if="errorMsg">{{ errorMsg }}</h3>
  </div>
    
</template>

<script>

import axios from 'axios'
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
  name: 'GetItemById',  
  data() {
    return {
      user: {
        id: '',
        name: ''
      },
      formData: {
        userId: '',
      },      
      errorMsg: '',
    }
  },
  methods: {
    async getItemsById() {
      try {
        const user = await userManager.getUser();
        
        // Use access_token instead of id_token for API authorization
        const token = user && user.access_token ? user.access_token : null;
        
        if (!token) {
          this.errorMsg = 'Not authenticated';
          return;
        }
        const response = await axios.get(
          process.env.VUE_APP_API_ENDPOINT + this.formData.userId,
          {
            headers: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
          }
        );
        this.user = response.data;
        this.errorMsg = '';
      } catch (error) {
        console.log(error);
        this.errorMsg = 'Error retrieving data';
      }
    },
  },
}
</script>

<style scoped>

</style>