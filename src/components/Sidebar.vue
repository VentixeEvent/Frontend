<script setup>
import { ref } from "vue";

// Define navigation items with Bootstrap icons
const navItems = ref([
  { name: "Dashboard", path: "/", icon: "bi-house-fill" },
  { name: "Bookings", path: "/about", icon: "bi-calendar-check" },
  { name: "Events", path: "/events", icon: "bi-flag" },
]);
</script>

<style>
@import "bootstrap-icons/font/bootstrap-icons.css";
</style>

<template>
  <div class="rounded-3 custom-bg d-flex flex-column h-100">
    <div class="p-4 d-flex flex-column h-100">
      <div
        class="sidebar-header d-flex justify-content-center align-items-center pb-3 mb-3 w-100"
      >
        <img src="../assets/logo.svg" alt="Logo" class="me-2" height="24" />
        <h5 class="h5-semibold brand-text brand-color mb-0">Ventixe</h5>
      </div>
      <ul class="nav nav-pills flex-column w-100 mb-3 gap-1">
        <li class="nav-item" v-for="item in navItems" :key="item.path">
          <RouterLink
            :to="item.path"
            class="nav-link d-flex align-items-center text-dark body-14"
            active-class="active bg-light-subtle body-14-semibold"
          >
            <i class="bi me-3 fs-6 custom-icon-color" :class="item.icon"></i>
            <span>{{ item.name }}</span>
          </RouterLink>
        </li>
      </ul>

      <!-- Mobile App Promotion Card, make it responsive -->
      <div class="card mb-4 border-0 shadow-sm app-promo-card">
        <div class="card-body p-3">
          <div class="d-flex align-items-center mb-2">
            <div
              class="app-icon me-3 d-flex align-items-center justify-content-center"
            >
              <i class="bi bi-phone fs-3 text-white"></i>
            </div>
            <div>
              <h6 class="title-16-semibold mb-0">Ventrixe Mobile</h6>
              <p class="body-12 text-muted">Take your work anywhere</p>
            </div>
          </div>

          <p class="body-12 mb-2">
            Get instant notifications, manage bookings on the go, and access
            your data even offline.
          </p>

          <div class="d-flex justify-content-between">
            <a
              href="#"
              class="btn btn-sm btn-outline-dark d-flex align-items-center btn-12-medium"
            >
              <i class="bi bi-apple me-1"></i> App Store
            </a>
            <a
              href="#"
              class="btn btn-sm btn-outline-dark d-flex align-items-center btn-12-medium"
            >
              <i class="bi bi-google-play me-1"></i> Google Play
            </a>
          </div>
        </div>
      </div>

      <!-- Spacer to push signout button to bottom -->
      <div class="flex-grow-1"></div>

      <!-- Bottom section for signout -->
      <div class="mt-2 pt-2 border-top">
        <button
          class="btn w-100 d-flex align-items-center justify-content-center gap-2 signout-btn btn-14-medium"
        >
          <i class="bi bi-box-arrow-right"></i>
          <span>Sign Out</span>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Minimal custom CSS - only for specific behaviors not covered by Bootstrap */
.router-link-active:not(.active) {
  background-color: rgba(0, 0, 0, 0.05);
}

/* Override Bootstrap nav-link padding for better spacing */
.nav-link {
  padding: 0.6rem 1rem;
  border-radius: 0.5rem;
  transition: all 0.2s;
}

.nav-link:hover {
  background-color: var(--color-hover);
}

.custom-icon-color {
  color: var(--secondary-100);
}

.brand-color {
  color: var(--secondary-110);
}

.custom-bg {
  background-color: var(--cool-grey-30);
}

/* App promo card styling */
.app-promo-card {
  background-color: var(--grey-20);
  min-height: 12rem;
  max-width: 300px;
}

.app-icon {
  width: 40px;
  height: 40px;
  background-color: var(--secondary-100);
  border-radius: 8px;
}

.signout-btn {
  background-color: var(--cool-grey-20);
  border-radius: 0.25rem;
}
</style>
