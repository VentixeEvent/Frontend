<script setup>
import { ref } from "vue";

// Sample breadcrumb data - in a real app, you'd likely compute this based on the current route
const breadcrumbs = ref([
  { name: "Home", path: "/" },
  { name: "Dashboard", path: "/dashboard", active: true },
]);
</script>

<template>
  <header class="bg-header p-3 rounded-3">
    <div class="d-flex flex-column">
      <!-- Breadcrumb navigation -->
      <nav aria-label="breadcrumb" class="mb-2">
        <ol class="breadcrumb m-0">
          <li
            class="breadcrumb-item"
            v-for="(crumb, index) in breadcrumbs"
            :key="index"
          >
            <router-link
              v-if="!crumb.active"
              :to="crumb.path"
              class="text-decoration-none"
            >
              {{ crumb.name }}
            </router-link>
            <span v-else class="fw-medium text-secondary">{{
              crumb.name
            }}</span>
          </li>
        </ol>
      </nav>

      <!-- Header content -->
      <div class="d-flex justify-content-between align-items-center">
        <h1 class="h4 mb-0 text-secondary-100">Dashboard</h1>

        <div>
          <div class="d-flex align-items-center gap-3">
            <!-- Search bar -->
            <div class="search-container">
              <div class="input-group">
                <span class="input-group-text bg-white border-end-0">
                  <i class="bi bi-search"></i>
                </span>
                <input
                  type="text"
                  class="form-control border-start-0 ps-0"
                  placeholder="Search..."
                  aria-label="Search"
                />
              </div>
            </div>

            <!-- Notification icon -->
            <div class="icon-circle">
              <i class="bi bi-bell"></i>
            </div>

            <!-- Settings icon -->
            <div class="icon-circle">
              <i class="bi bi-gear"></i>
            </div>

            <!-- Profile -->
            <div class="d-flex align-items-center ms-3">
              <div
                class="profile-icon rounded-circle bg-secondary text-white d-flex justify-content-center align-items-center me-2"
              >
                <span>JD</span>
              </div>
              <span>John Doe</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </header>
</template>

<style scoped>
header {
  width: 100%;
  background-color: var(--grey-20);
}

.search-container {
  display: flex;
  width: 307px;
  height: 40px;
  padding: 9px 16px;
  align-items: center;
  gap: 6px;
}

.icon-circle {
  width: 38px;
  height: 38px;
  background-color: var(--secondary-100);
  border-radius: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  color: var(--grey-10);
  cursor: pointer;
}

.profile-icon {
  width: 36px;
  height: 36px;
  font-size: 0.875rem;
  font-weight: bold;
}

/* Remove default box-shadow on form controls when focused */
.form-control:focus {
  box-shadow: none;
  border-color: var(--grey-40);
}

/* Breadcrumb styling to match your design */
.breadcrumb {
  padding: 0;
  font-size: 0.85rem;
}

.breadcrumb-item + .breadcrumb-item::before {
  color: var(--grey-60);
}

.breadcrumb a {
  color: var(--primary-100);
}

.breadcrumb a:hover {
  color: var(--secondary-100);
}

.text-secondary-100 {
  color: var(--secondary-100);
}
</style>
