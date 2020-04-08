<template>
  <div>
    <nav class="w-full p-6 bg-gray-100 fixed flex justify-between z-10 top-0 shadow">
      <div class="w-1/2">
        <!-- <img class="logo" src="../../assets/logo.svg" alt="Logo" /> -->
        <span id="title">TAC Chat Integration With CometChat</span>
      </div>
      <div class="w-1/2">
        <div class="flex">
          <span class="inline-block mr-2">
            Welcome,
            <b>{{ name }}</b>
          </span>
          <img v-bind:src="avatar" class="h-6 w-6 inline-block" alt="User avatar" />
        </div>
      </div>
    </nav>
  </div>
</template>

<script>
export default {
  props: ["name", "avatar"]
};
</script>
