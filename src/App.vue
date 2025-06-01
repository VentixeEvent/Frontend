<script setup>
import { RouterLink, RouterView } from "vue-router";
import Sidebar from "./components/Sidebar.vue";
import Header from "./components/Header.vue";
import Footer from "./components/Footer.vue";
</script>

<template>
  <div class="d-flex pt-2 ps-2 pe-2 pb-2">
    <!-- Sidebar with adjusted height to account for padding -->
    <div class="flex-shrink-0 d-flex flex-column h-100">
      <Sidebar class="flex-grow-1" />
    </div>

    <!-- Main content area -->
    <div class="flex-grow-1 ps-2 d-flex flex-column h-100">
      <Header />
      <main class="flex-grow-1 mt-2 d-flex flex-column mb-0">
        <RouterView class="flex-grow-1" />
      </main>
      <Footer class="mt-0" />
    </div>
  </div>
</template>

<style scoped>
.d-flex {
  height: calc(100vh - 16px);
  overflow: hidden;
}

main {
  display: flex;
  flex-direction: column;
  min-height: 0; /* Allows flex items to shrink below content height */
}
</style>
