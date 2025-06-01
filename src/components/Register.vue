<script setup>
import { ref, computed } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();

const form = ref({
  fullName: "",
  email: "",
  password: "",
  confirmPassword: "",
  agreeTerms: false,
});

const showPassword = ref(false);
const showConfirmPassword = ref(false);
const isSubmitting = ref(false);
const errorMessage = ref("");

const passwordStrength = computed(() => {
  const password = form.value.password;
  if (!password) return 0;

  let score = 0;
  // Length check
  if (password.length >= 8) score += 25;
  // Contains lowercase
  if (/[a-z]/.test(password)) score += 25;
  // Contains uppercase
  if (/[A-Z]/.test(password)) score += 25;
  // Contains number or special char
  if (/[0-9!@#$%^&*(),.?":{}|<>]/.test(password)) score += 25;

  return score;
});

const passwordStrengthText = computed(() => {
  const score = passwordStrength.value;
  if (score === 0) return "";
  if (score <= 25) return "Weak";
  if (score <= 50) return "Fair";
  if (score <= 75) return "Good";
  return "Strong";
});

const passwordStrengthColor = computed(() => {
  const score = passwordStrength.value;
  if (score === 0) return "";
  if (score <= 25) return "bg-danger";
  if (score <= 50) return "bg-warning";
  if (score <= 75) return "bg-info";
  return "bg-success";
});

const passwordsMatch = computed(() => {
  return (
    form.value.password === form.value.confirmPassword &&
    form.value.password !== ""
  );
});

const isFormValid = computed(() => {
  return (
    form.value.fullName.trim() !== "" &&
    form.value.email.includes("@") &&
    form.value.password.length >= 8 &&
    passwordsMatch.value &&
    form.value.agreeTerms
  );
});

const togglePasswordVisibility = (field) => {
  if (field === "password") {
    showPassword.value = !showPassword.value;
  } else {
    showConfirmPassword.value = !showConfirmPassword.value;
  }
};

const handleRegister = async () => {
  if (!isFormValid.value) {
    errorMessage.value = "Please fill all fields correctly.";
    return;
  }

  try {
    isSubmitting.value = true;
    errorMessage.value = "";

    // Here you would typically handle registration with your backend
    // For example: await authService.register(form.value)

    console.log("Registration attempted with:", form.value);

    // Simulate API delay
    await new Promise((resolve) => setTimeout(resolve, 1500));

    // Redirect to login or dashboard
    router.push("/login");
  } catch (error) {
    errorMessage.value = "Registration failed. Please try again.";
    console.error("Registration error:", error);
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<template>
  <div class="register-container">
    <div class="row justify-content-center">
      <div class="col-md-10 col-lg-8 col-xl-6">
        <div class="auth-card shadow-sm">
          <!-- Logo and Title -->
          <div class="text-center mb-4 pt-2">
            <div class="logo-container mb-3">
              <img src="../assets/logo.svg" alt="Ventrixe Logo" height="128" />
            </div>
            <h1 class="h3-semibold text-secondary-110">Create Account</h1>
            <p class="body-14 text-muted">
              Join Ventrixe and start managing your events
            </p>
          </div>

          <!-- Error Message -->
          <div v-if="errorMessage" class="alert alert-danger body-14 mb-4">
            {{ errorMessage }}
          </div>

          <!-- Registration Form -->
          <form @submit.prevent="handleRegister">
            <div class="mb-3">
              <label for="fullName" class="form-label body-14-semibold"
                >Full Name</label
              >
              <div class="input-group">
                <span class="input-group-text bg-white border-end-0">
                  <i class="bi bi-person"></i>
                </span>
                <input
                  type="text"
                  class="form-control border-start-0"
                  id="fullName"
                  placeholder="John Doe"
                  v-model="form.fullName"
                  required
                />
              </div>
            </div>

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
                  v-model="form.email"
                  required
                />
              </div>
            </div>

            <div class="mb-3">
              <label for="password" class="form-label body-14-semibold"
                >Password</label
              >
              <div class="input-group">
                <span class="input-group-text bg-white border-end-0">
                  <i class="bi bi-lock"></i>
                </span>
                <input
                  :type="showPassword ? 'text' : 'password'"
                  class="form-control border-start-0 border-end-0"
                  id="password"
                  placeholder="Create a secure password"
                  v-model="form.password"
                  required
                  autocomplete="new-password"
                />
                <button
                  class="input-group-text bg-white border-start-0"
                  type="button"
                  @click="togglePasswordVisibility('password')"
                >
                  <i
                    :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
                  ></i>
                </button>
              </div>

              <div v-if="form.password" class="mt-2">
                <div class="progress" style="height: 5px">
                  <div
                    class="progress-bar"
                    :class="passwordStrengthColor"
                    role="progressbar"
                    :style="{ width: passwordStrength + '%' }"
                    :aria-valuenow="passwordStrength"
                    aria-valuemin="0"
                    aria-valuemax="100"
                  ></div>
                </div>
                <div class="d-flex justify-content-between mt-1">
                  <span class="body-12 text-muted">Password strength:</span>
                  <span
                    class="body-12-semibold"
                    :class="{
                      'text-danger': passwordStrength <= 25,
                      'text-warning':
                        passwordStrength > 25 && passwordStrength <= 50,
                      'text-info':
                        passwordStrength > 50 && passwordStrength <= 75,
                      'text-success': passwordStrength > 75,
                    }"
                  >
                    {{ passwordStrengthText }}
                  </span>
                </div>
              </div>
            </div>

            <div class="mb-4">
              <label for="confirmPassword" class="form-label body-14-semibold"
                >Confirm Password</label
              >
              <div class="input-group">
                <span class="input-group-text bg-white border-end-0">
                  <i class="bi bi-lock-fill"></i>
                </span>
                <input
                  :type="showConfirmPassword ? 'text' : 'password'"
                  class="form-control border-start-0 border-end-0"
                  id="confirmPassword"
                  placeholder="Confirm your password"
                  v-model="form.confirmPassword"
                  required
                  autocomplete="new-password"
                  :class="{
                    'is-invalid': form.confirmPassword && !passwordsMatch,
                    'is-valid': form.confirmPassword && passwordsMatch,
                  }"
                />
                <button
                  class="input-group-text bg-white border-start-0"
                  type="button"
                  @click="togglePasswordVisibility('confirm')"
                >
                  <i
                    :class="
                      showConfirmPassword ? 'bi bi-eye-slash' : 'bi bi-eye'
                    "
                  ></i>
                </button>
              </div>
              <div
                v-if="form.confirmPassword && !passwordsMatch"
                class="invalid-feedback d-block"
              >
                Passwords do not match
              </div>
            </div>

            <div class="mb-4">
              <div class="form-check">
                <input
                  type="checkbox"
                  class="form-check-input"
                  id="agreeTerms"
                  v-model="form.agreeTerms"
                  required
                />
                <label class="form-check-label body-12" for="agreeTerms">
                  I agree to the
                  <a href="#" class="text-primary">Terms of Service</a> and
                  <a href="#" class="text-primary">Privacy Policy</a>
                </label>
              </div>
            </div>

            <button
              type="submit"
              class="btn btn-primary w-100 mb-3 py-2 body-14-semibold"
              :disabled="isSubmitting || !isFormValid"
            >
              <span
                v-if="isSubmitting"
                class="spinner-border spinner-border-sm me-2"
                role="status"
              ></span>
              {{ isSubmitting ? "Creating your account..." : "Create Account" }}
            </button>

            <!-- Social Registration -->
            <div class="social-login">
              <div class="divider mb-3">
                <span class="or-text body-12">or register with</span>
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

          <!-- Login Link -->
          <div class="text-center body-14 mt-4">
            Already have an account?
            <router-link
              to="/login"
              class="text-primary text-decoration-none ms-1"
            >
              Sign In
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.register-container {
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

.btn-primary:hover:not(:disabled) {
  background-color: var(--secondary-110);
  border-color: var(--secondary-110);
}

.btn-primary:disabled {
  background-color: var(--secondary-80);
  border-color: var(--secondary-80);
  opacity: 0.7;
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
