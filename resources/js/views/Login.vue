<template>
  <div class="min-h-screen bg-gray-50 flex flex-col justify-center py-12 sm:px-6 lg:px-8">
    <div class="sm:mx-auto sm:w-full sm:max-w-md">
      <h2 class="mt-6 text-center text-3xl leading-9 font-extrabold text-gray-900">Sign in</h2>
    </div>

    <div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
      <div class="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10">
        <form v-on:submit.prevent="authLoginAppUser">
          <div>
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
            <span class="block w-full rounded-md shadow-sm">
              <button
                type="submit"
                class="w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-500 focus:outline-none focus:border-indigo-700 focus:shadow-outline-indigo active:bg-indigo-700 transition duration-150 ease-in-out"
              >
                Sign in
                <span v-if="showSpinner" class="fa fa-spin fa-spinner"></span>
              </button>
            </span>
          </div>
        </form>
        <div class="text-center mt-50 link-reg mt-6">
          <p v-on:click="redirectToRegister" class="pointer">
            Don't have an account yet?
            <span class="text-blue-500">Register</span>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { CometChat } from "@cometchat-pro/chat";
export default {
  data() {
    return {
      username: "",
      password: '',
      showSpinner: false,
      token: '',
    };
  },
  methods: {
    authLoginAppUser() {
      let userData = {
        username: this.username,
        password: this.password
      };

      if (this.username && this.password) {
        axios.post(`http://cometchat-sandbox.test/api/login`, userData).then(response => {
          this.logUserInToCometChat(response.data.token)
        }).catch(error => {
          alert(error.response.data.message);
          console.log(error.response.data.message);
        })
      } else {
        alert('Please check your credentials');
      }
    },
    redirectToRegister() {
      this.$router.push({ name: 'register' });
    },
    logUserInToCometChat(token) {
      this.showSpinner = true;
      CometChat.login(token).then(
        () => {
          this.showSpinner = false;
          console.log("successfully login user");
          this.$router.push({
            name: 'chat',
            params: { username: this.username, authenticated: true }
          });
        },
        error => {
          this.showSpinner = false;
          alert("Whops. Something went wrong. This commonly happens when you enter a username that doesn't exist. Check the console for more information");
          console.log("Login failed with error:", error.code);
        }
      );
    },
  }
};
</script>
