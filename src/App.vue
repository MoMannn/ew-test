<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
</script>


<template>
  <header>
    <h1 class="page-title">Welcome to Apillon Wallet Demo</h1>
    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>
</template>

<style>
.page-title {
  text-align: left; /* Changed from center to left */
  margin: 0;
  padding: 1rem 2rem; /* Added horizontal padding */
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background-color: white;
  z-index: 99;
  font-size: 1.5rem; /* Optional: reduced font size */
}

.wrapper {
  margin-top: 5rem; /* Add space below the fixed header */
}
</style>