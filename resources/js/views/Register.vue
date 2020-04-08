<template>
  <div class="min-h-screen bg-gray-50 flex flex-col justify-center py-12 sm:px-6 lg:px-8">
    <div class="sm:mx-auto sm:w-full sm:max-w-md">
      <h2 class="mt-6 text-center text-3xl leading-9 font-extrabold text-gray-900">Create an account</h2>
    </div>

    <div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
      <div class="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10">
        <form v-on:submit.prevent="registerAppUser">
          <div class="mt-6">
            <label for="username" class="block text-sm font-medium leading-5 text-gray-700">Username</label>
            <div class="mt-1 rounded-md shadow-sm">
              <input
                id="username"
                type="text"
                v-model="username"
                class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md placeholder-gray-400 focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
              />
            </div>
          </div>

          <div class="mt-6">
            <label for="password" class="block text-sm font-medium leading-5 text-gray-700">Password</label>
            <div class="mt-1 rounded-md shadow-sm">
              <input
                id="password"
                type="password"
                v-model="password"
                placeholder="******"
                class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md placeholder-gray-400 focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
              />
            </div>
          </div>
          <div class="mt-6">
            <label
              for="password_confirmation"
              class="block text-sm font-medium leading-5 text-gray-700"
            >Confirm Password</label>
            <div class="mt-1 rounded-md shadow-sm">
              <input
                id="password_confirmation"
                type="password"
                v-model="password_confirmation"
                placeholder="******"
                class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md placeholder-gray-400 focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
              />
            </div>
          </div>

          <div class="mt-6">
            <span class="block w-full rounded-md shadow-sm">
              <button
                type="submit"
                class="w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-500 focus:outline-none focus:border-indigo-700 focus:shadow-outline-indigo active:bg-indigo-700 transition duration-150 ease-in-out"
              >
                Sign up
                <span v-if="showSpinner" class="fa fa-spin fa-spinner"></span>
              </button>
            </span>
          </div>
          <div class="text-center mt-50 link-reg mt-6">
            <p v-on:click="redirectToLogin">
              Already have an account?
              <span class="text-blue-500">Login</span>
            </p>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: "",
      password: '',
      password_confirmation: '',
      showSpinner: false,
    };
  },
  methods: {
    registerAppUser() {
      if (this.username && this.password && this.password_confirmation) {
        if (this.password && this.password_confirmation) {
          let userData = {
            username: this.username,
            password: this.password,
            password_confirmation: this.password_confirmation
          };

          axios.post(`http://cometchat-sandbox.test/api/register`, userData)
            .then(response => {
              if (response.data) {
                this.createUserOnCometChat(this.username);
              }
            }).catch(error => {
              alert(error.response.data.message);
            })
        }
      }
    },
    redirectToLogin() {
      this.$router.push({ name: 'login' })
    },
    async createUserOnCometChat(username) {
      let url = `https://api-eu.cometchat.io/v2.0/users`;
      let data = {
        uid: username,
        name: `${username} sample`,
        avatar: 'https://data-eu.cometchat.io/assets/images/avatars/captainamerica.png',
      };

      try {
        const userResponse = await fetch(url, {
          method: 'POST',
          headers: new Headers({
            appid: process.env.MIX_COMMETCHAT_APP_ID,
            apikey: process.env.MIX_COMMETCHAT_API_KEY,
            'Content-Type': 'application/json'
          }),
          body: JSON.stringify(data),
        });
        const userJson = await userResponse.json();

        console.log('New User', userJson);
        this.createAuthTokenAndSaveForUser(username);
        this.redirectToLogin();
      } catch (error) {
        console.log('Error', error);
      }
    },
    async createAuthTokenAndSaveForUser(uid) {
      let url = `https://api-eu.cometchat.io/v2.0/users/${uid}/auth_tokens`;

      try {
        const tokenResponse = await fetch(url, {
          method: 'POST',
          headers: new Headers({
            appid: process.env.MIX_COMMETCHAT_APP_ID,
            apikey: process.env.MIX_COMMETCHAT_API_KEY,
            'Content-Type': 'application/json'
          }),
        });
        const tokenJSON = await tokenResponse.json();
        this.addUserToAGroup(uid);
        this.sendTokenToServer(tokenJSON.data.authToken, tokenJSON.data.uid);
      } catch (error) {
        console.log('Error Token', error);
      }
    },
    async addUserToAGroup(uid) {
      let url = `https://api-eu.cometchat.io/v2.0/groups/${process.env.MIX_COMMETCHAT_GUID}/members`;
      let data = {
        "participants": [uid]
      };

      try {
        const groupResponse = await fetch(url, {
          method: 'POST',
          headers: new Headers({
            appid: process.env.MIX_COMMETCHAT_APP_ID,
            apikey: process.env.MIX_COMMETCHAT_API_KEY,
            'Content-Type': 'application/json'
          }),
          body: JSON.stringify(data),
        });
        const groupJson = await groupResponse.json();

        console.log('Added to group', groupJson);
      } catch (error) {
        console.log('Error', error);
      }
    },
    sendTokenToServer(token, uid) {
      axios.post(`http://cometchat-sandbox.test/api/update/token`, { token, uid })
        .then(response => {
          console.log("Token updated successfully", response);
        }).catch(error => {
          alert(error.response.data.message);
        })
    }
  }
}
</script>

<style>
</style>