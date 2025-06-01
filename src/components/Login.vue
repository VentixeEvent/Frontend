<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const email = ref("");
const password = ref("");
const rememberMe = ref(false);
const showPassword = ref(false);
const isSubmitting = ref(false);
const errorMessage = ref("");

const handleLogin = async () => {
  try {
    isSubmitting.value = true;
    errorMessage.value = "";

    // Here you would typically handle authentication with your backend
    // For example: await authService.login(email.value, password.value)

    console.log("Login attempted with:", {
      email: email.value,
      password: password.value,
      rememberMe: rememberMe.value,
    });

    // Simulate API delay
    await new Promise((resolve) => setTimeout(resolve, 1000));

    // Redirect to dashboard on successful login
    router.push("/");
  } catch (error) {
    errorMessage.value = "Invalid email or password. Please try again.";
    console.error("Login error:", error);
  } finally {
    isSubmitting.value = false;
  }
};

const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};
</script>

<template>
  <div class="login-container">
    <div class="row justify-content-center">
      <div class="col-md-10 col-lg-8 col-xl-6">
        <div class="auth-card shadow-sm">
          <!-- Logo and Title -->
          <div class="text-center mb-4 pt-3">
            <div class="logo-container mb-3">
              <img src="../assets/logo.svg" alt="Ventrixe Logo" height="128" />
            </div>
            <h1 class="h3-semibold text-secondary-110">Welcome Back</h1>
            <p class="body-14 text-muted">Sign in to continue with Ventrixe</p>
          </div>

          <!-- Error Message -->
          <div v-if="errorMessage" class="alert alert-danger body-14 mb-4">
            {{ errorMessage }}
          </div>

          <!-- Login Form -->
          <form @submit.prevent="handleLogin" class="mb-4">
            <div class="mb-3">
              <label for="email" class="form-label body-14-semibold"
                >Email Address</label
              >
              <div class="input-group">
                <span class="input-group-text bg-white border-end-0">
                  <i class="bi bi-envelope"></i>
                </span>
                <input
                  type="email"
                  class="form-control border-start-0"
                  id="email"
                  placeholder="your.email@example.com"
                  v-model="email"
                  required
                  autocomplete="email"
                />
              </div>
            </div>

            <div class="mb-3">
              <div
                class="d-flex justify-content-between align-items-center mb-1"
              >
                <label for="password" class="form-label body-14-semibold mb-0"
                  >Password</label
                >
                <router-link
                  to="/forgot-password"
                  class="body-12 text-decoration-none"
                >
                  Forgot Password?
                </router-link>
              </div>
              <div class="input-group">
                <span class="input-group-text bg-white border-end-0">
                  <i class="bi bi-lock"></i>
                </span>
                <input
                  :type="showPassword ? 'text' : 'password'"
                  class="form-control border-start-0 border-end-0"
                  id="password"
                  placeholder="Enter your password"
                  v-model="password"
                  required
                  autocomplete="current-password"
                />
                <button
                  class="input-group-text bg-white border-start-0"
                  type="button"
                  @click="togglePasswordVisibility"
                >
                  <i
                    :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
                  ></i>
                </button>
              </div>
            </div>

            <div class="d-flex justify-content-between mb-4">
              <div class="form-check">
                <input
                  type="checkbox"
                  class="form-check-input"
                  id="rememberMe"
                  v-model="rememberMe"
                />
                <label class="form-check-label body-12" for="rememberMe">
                  Remember me
                </label>
              </div>
            </div>

            <button
              type="submit"
              class="btn btn-primary w-100 mb-3 py-2 body-14-semibold"
              :disabled="isSubmitting"
            >
              <span
                v-if="isSubmitting"
                class="spinner-border spinner-border-sm me-2"
                role="status"
              ></span>
              {{ isSubmitting ? "Signing in..." : "Sign In" }}
            </button>

            <!-- Social Login Buttons -->
            <div class="social-login">
              <div class="divider mb-3">
                <span class="or-text body-12">or continue with</span>
              </div>

              <div class="d-flex justify-content-center gap-3">
                <button
                  type="button"
                  class="btn btn-outline-secondary social-btn"
                >
                  <i class="bi bi-google"></i>
                </button>
                <button
                  type="button"
                  class="btn btn-outline-secondary social-btn"
                >
                  <i class="bi bi-apple"></i>
                </button>
                <button
                  type="button"
                  class="btn btn-outline-secondary social-btn"
                >
                  <i class="bi bi-microsoft"></i>
                </button>
              </div>
            </div>
          </form>

          <!-- Register Link -->
          <div class="text-center body-14 mb-3">
            Don't have an account?
            <router-link
              to="/register"
              class="text-primary text-decoration-none ms-1"
            >
              Create Account
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-container {
  background-color: var(--grey-20);
  padding: 2rem;
}

.auth-card {
  background-color: var(--grey-10);
  border-radius: 1rem;
  padding: 2rem;
  width: 100%;
}

.btn-primary {
  background-color: var(--secondary-100);
  border-color: var(--secondary-100);
  transition: all 0.2s;
}

.btn-primary:hover {
  background-color: var(--secondary-110);
  border-color: var(--secondary-110);
}

.text-primary {
  color: var(--secondary-100) !important;
}

.form-control:focus,
.form-check-input:focus {
  border-color: var(--secondary-60);
  box-shadow: 0 0 0 0.25rem rgba(55, 67, 125, 0.25);
}

.form-check-input:checked {
  background-color: var(--secondary-100);
  border-color: var(--secondary-100);
}

.divider {
  display: flex;
  align-items: center;
  text-align: center;
  color: var(--grey-60);
}

.divider::before,
.divider::after {
  content: "";
  flex: 1;
  border-bottom: 1px solid var(--grey-40);
}

.or-text {
  padding: 0 1rem;
  color: var(--grey-60);
}

.social-btn {
  width: 48px;
  height: 48px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.25rem;
  transition: all 0.2s;
}

.social-btn:hover {
  background-color: var(--grey-20);
  border-color: var(--secondary-80);
  color: var(--secondary-100);
}

.logo-container {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
