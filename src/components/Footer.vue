<script setup>
import { ref } from "vue";

const currentYear = new Date().getFullYear();
const links = ref([
  { name: "Terms", url: "/terms" },
  { name: "Privacy", url: "/privacy" },
  { name: "Support", url: "/support" },
]);
</script>

<template>
  <footer class="mt-3 py-3 bg-light rounded-3">
    <div class="container-fluid">
      <div class="d-flex flex-wrap justify-content-between align-items-center">
        <!-- Copyright section -->
        <div class="col-md-4 mb-md-0 mb-2">
          <p class="mb-0 text-muted small">
            &copy; {{ currentYear }} Ventrixe. All rights reserved.
          </p>
        </div>

        <!-- Footer links -->
        <div class="col-md-4 d-flex justify-content-end">
          <ul class="nav">
            <li class="nav-item" v-for="link in links" :key="link.name">
              <a :href="link.url" class="nav-link px-2 text-muted small">{{
                link.name
              }}</a>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </footer>
</template>

<style scoped>
footer {
  width: 100%;
  background-color: #f7f7f7 !important;
}

.nav-link {
  color: #37437d;
  transition: all 0.2s;
}

.nav-link:hover {
  color: #232952;
}
</style>
