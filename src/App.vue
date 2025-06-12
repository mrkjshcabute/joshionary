<script setup>
 
</script>

<template>
  <div>
    <router-link to="/"></router-link>
    <router-link to="/search"></router-link>
  </div>
  <transition name="fade">
    <router-view />
  </transition>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
