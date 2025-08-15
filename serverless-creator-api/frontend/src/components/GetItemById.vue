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
  authority: process.env.VUE_APP_COGNITO_AUTHORITY,
  client_id: process.env.VUE_APP_COGNITO_CLIENT_ID,
  redirect_uri:  window.location.origin,
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
        console.log(token)
        const idToken =  user && user.id_token ? user.id_token : null;
        if (!idToken) {
          this.errorMsg = 'Not authenticated';
          return;
        }
        const response = await axios.get(
          process.env.VUE_APP_API_ENDPOINT + this.formData.userId,
          {
            headers: {
              'Authorization': `Bearer ${idToken}`,
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