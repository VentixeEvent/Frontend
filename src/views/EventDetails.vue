<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";

import VenueSvg from "../assets/venue.svg"; // Import the venue SVG

const route = useRoute();
const router = useRouter();

const API_BASE_URL =
  "https://eventservice01-hugcdba0e9g5ewbk.swedencentral-01.azurewebsites.net";

// Event data
const event = ref(null);
const isLoading = ref(false);
const error = ref(null);

// UI state
const selectedPackage = ref(null);
const showSeatPlan = ref(true);

const getEventIdentifier = (slug) => {
  // If slug is numeric, treat it as ID for backward compatibility
  if (/^\d+$/.test(slug)) {
    return { type: "id", value: slug };
  }
  // Otherwise, it's a slug
  return { type: "slug", value: slug };
};

// Fetch event details by ID or slug
const fetchEventDetails = async (identifier) => {
  try {
    isLoading.value = true;
    error.value = null;

    const { type, value } = getEventIdentifier(identifier);

    // First, fetch all events to find the one matching our identifier
    const eventsResponse = await fetch(`${API_BASE_URL}/api/events`);
    if (!eventsResponse.ok) {
      throw new Error("Failed to fetch events");
    }

    const data = await eventsResponse.json();
    console.log("API Response:", data);

    // Extract events array - handle both direct array and nested result
    let eventsArray = [];
    if (Array.isArray(data)) {
      eventsArray = data;
    } else if (Array.isArray(data.result)) {
      eventsArray = data.result;
    } else {
      console.warn("Unexpected API response structure:", data);
      throw new Error("Invalid response format");
    }

    let targetEvent;

    if (type === "id") {
      // Find by ID
      targetEvent = eventsArray.find((e) => e.id.toString() === value);
    } else {
      // Find by slug (generated from title)
      targetEvent = eventsArray.find((e) => {
        const eventSlug = e.title
          .toLowerCase()
          .replace(/[^\w\s-]/g, "")
          .replace(/\s+/g, "-")
          .replace(/-+/g, "-")
          .trim("-");
        return eventSlug === value;
      });
    }

    if (!targetEvent) {
      throw new Error("Event not found");
    }

    // Transform the API data to include additional UI properties
    event.value = {
      ...targetEvent, // ✅ Fixed: use targetEvent instead of data
      ticketsLeft:
        targetEvent.ticketsLeft || Math.floor(Math.random() * 25000) + 5000,
      price: targetEvent.price || Math.floor(Math.random() * 300) + 50,
      formattedDate: formatDate(targetEvent.eventDate),
      formattedTime: formatTime(targetEvent.eventDate),
      vipLounge: true,
      backstageAccess: true,
      seatPlan: {
        sections: [
          { name: "General", price: 80, color: "#E8E8E8" },
          { name: "VIP Gold", price: 150, color: "#FFD700" },
          { name: "VIP Silver", price: 120, color: "#C0C0C0" },
          { name: "Bronze Section", price: 100, color: "#CD7F32" },
          { name: "General Admission", price: 80, color: "#A9A9A9" },
          { name: "Backstage Access", price: 300, color: "#FF69B4" },
          { name: "VIP Lounge", price: 200, color: "#9370DB" },
        ],
      },
      packages: [
        {
          name: "General Admission Package",
          description: "Access to general seating area",
          features: ["Standing", "Access to Festival Grounds"],
          price: 80,
        },
        {
          name: "Silver Package",
          description: "Enhanced festival experience",
          features: ["Seating", "All-Day View"],
          price: 120,
        },
        {
          name: "Gold Package",
          description: "Premium festival experience",
          features: ["Seating", "Prime View"],
          price: 150,
        },
        {
          name: "Platinum Package",
          description: "Ultimate VIP experience",
          features: ["Seating", "Best Stage"],
          price: 190,
        },
        {
          name: "Diamond Package",
          description: "Exclusive premium experience",
          features: ["Seating", "Front Row View"],
          price: 250,
        },
        {
          name: "VIP Lounge Package",
          description: "Exclusive lounge access",
          features: ["Seating", "Exclusive Lounge"],
          price: 155,
        },
        {
          name: "Artist Meet-and-Greet Package",
          description: "Meet your favorite artists",
          features: ["Standing", "Backstage Access"],
          price: 180,
        },
        {
          name: "Ultimate Access Package",
          description: "Complete festival experience",
          features: ["Standing", "All Inclusive Benefits"],
          price: 300,
        },
      ],
      merchandise: [
        {
          name: "Event Cap",
          price: 25,
          image: "../assets/merch-cap.jpg",
        },
        {
          name: "Festival T-Shirt",
          price: 35,
          image: "../assets/merch-tshirt.jpg",
        },
        {
          name: "Light-up Wristband",
          price: 15,
          image: "../assets/merch-wristband.jpg",
        },
      ],
      termsAndConditions: [
        "Ticket Purchases and Entry",
        "All attendees must possess a valid ticket for entry",
        "Tickets are non-refundable and non-transferable unless specified by the event organizer",
        "Attendees must present a valid government issued ID along with their ticket at the gate",
        "Security and Safety",
        "Attendees are subject to security checks, including bag inspections, upon entry",
        "Prohibited items include weapons, illegal substances, glass containers, and other hazardous materials",
        "The event organizer reserves the right to refuse entry to individuals deemed a security risk",
        "Code of Conduct",
        "Attendees are expected to behave respectfully and responsibly toward other attendees",
        "Any disruptive behavior, harassment, or illegal activity will result in immediate removal from the event without refund",
        "The event schedule is subject to change without prior notice",
      ],
    };
  } catch (err) {
    console.error("Error fetching event details:", err);
    error.value = err.message || "Failed to load event details";
  } finally {
    isLoading.value = false;
  }
};

// Helper functions (reuse from Events.vue)
const formatDate = (dateString) => {
  if (!dateString) return "";
  const date = new Date(dateString);
  return date.toLocaleDateString("en-US", {
    month: "short",
    day: "numeric",
    year: "numeric",
  });
};

const formatTime = (dateString) => {
  if (!dateString) return "";
  const date = new Date(dateString);
  return date.toLocaleTimeString("en-US", {
    hour: "numeric",
    minute: "2-digit",
    hour12: true,
  });
};

// Navigation functions
const goBack = () => {
  router.push("/events");
};

const shareEvent = async () => {
  if (navigator.share && event.value) {
    try {
      await navigator.share({
        title: event.value.title,
        text: event.value.description,
        url: window.location.href,
      });
    } catch (err) {
      console.log("Share cancelled");
    }
  }
};

// Package selection
const selectPackage = (pkg) => {
  selectedPackage.value = pkg;
};

const bookPackage = () => {
  if (selectedPackage.value) {
    // Implement booking functionality
    alert(
      `Booking ${selectedPackage.value.name} for $${selectedPackage.value.price}`
    );
  }
};

// Initialize component
onMounted(() => {
  const slug = route.params.slug; // Using slug parameter
  if (slug) {
    fetchEventDetails(slug);
  } else {
    error.value = "Event identifier not provided";
  }
});
</script>

<template>
  <div class="event-details-container">
    <!-- Header with Navigation -->
    <div class="event-header">
      <div class="header-controls">
        <button class="btn-back" @click="goBack">
          <i class="bi bi-chevron-left"></i>
        </button>
        <span class="header-title body-14-semibold">Event Details</span>
        <div class="header-actions">
          <button class="btn-action" @click="shareEvent">
            <i class="bi bi-share"></i>
          </button>
          <button class="btn-action">
            <i class="bi bi-three-dots"></i>
          </button>
        </div>
      </div>
    </div>

    <!-- Scrollable Content Area -->
    <div class="event-details-content">
      <!-- Loading State -->
      <div v-if="isLoading" class="loading-state">
        <div class="loading-content">
          <div class="spinner-modern"></div>
          <h5 class="h5-semibold mt-3">Loading Event Details</h5>
          <p class="body-14 text-muted">Fetching event information...</p>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="error-state">
        <div class="error-content">
          <div class="error-icon">
            <i class="bi bi-exclamation-triangle"></i>
          </div>
          <h5 class="h5-semibold mt-3">Event Not Found</h5>
          <p class="body-14 text-muted mb-4">{{ error }}</p>
          <button class="btn btn-primary modern-btn" @click="goBack">
            <i class="bi bi-arrow-left me-2"></i>
            Back to Events
          </button>
        </div>
      </div>

      <!-- Main Content -->
      <div v-else-if="event" class="event-content">
        <!-- Left Column -->
        <div class="event-main">
          <!-- Event Hero Section -->
          <div class="event-hero">
            <div class="event-image-container">
              <img
                v-if="event.image && event.image !== 'placeholder.jpg'"
                :src="event.image"
                :alt="event.title"
                class="event-image"
              />
              <div v-else class="event-placeholder">
                <div class="placeholder-content">
                  <i class="bi bi-calendar-event placeholder-icon"></i>
                  <span class="placeholder-text">Event Image</span>
                </div>
              </div>
              <div class="event-overlay">
                <div class="event-info">
                  <h1 class="event-title h2-semibold">{{ event.title }}</h1>
                  <div class="event-meta">
                    <div class="meta-item">
                      <i class="bi bi-calendar"></i>
                      <span
                        >{{ event.formattedDate }} •
                        {{ event.formattedTime }}</span
                      >
                    </div>
                    <div class="meta-item">
                      <i class="bi bi-geo-alt"></i>
                      <span>{{ event.location }}</span>
                    </div>
                  </div>
                  <div class="event-stats">
                    <div class="stat-item">
                      <span class="stat-value h3-semibold">{{
                        event.ticketsLeft.toLocaleString()
                      }}</span>
                      <span class="stat-label body-12">Tickets Left</span>
                    </div>
                    <div class="stat-item">
                      <span class="stat-value price h3-semibold"
                        >${{ event.price }}</span
                      >
                      <span class="stat-label body-12">Starting from</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- About Event Section -->
          <div class="section-card">
            <h3 class="section-title h4-semibold">About Event</h3>
            <p class="event-description body-14">{{ event.description }}</p>
          </div>

          <!-- Terms & Conditions -->
          <div class="section-card">
            <div class="section-header">
              <h3 class="section-title h4-semibold">Terms & Conditions</h3>
              <button class="btn-expand">
                <i class="bi bi-chevron-down"></i>
              </button>
            </div>
            <div class="terms-list">
              <div
                v-for="(term, index) in event.termsAndConditions"
                :key="index"
                class="term-item body-14"
              >
                {{ index + 1 }}. {{ term }}
              </div>
            </div>
          </div>

          <!-- Official Merchandise -->
          <div class="section-card">
            <h3 class="section-title h4-semibold">Official Merchandise</h3>
            <div class="merchandise-grid">
              <div
                v-for="item in event.merchandise"
                :key="item.name"
                class="merch-item"
              >
                <div class="merch-image">
                  <div class="merch-placeholder">
                    <i class="bi bi-bag"></i>
                  </div>
                </div>
                <div class="merch-info">
                  <h5 class="merch-name body-14-semibold">{{ item.name }}</h5>
                  <span class="merch-price body-12">${{ item.price }}</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Our Partners -->
          <div class="section-card">
            <h3 class="section-title h4-semibold">Our Partners</h3>
            <div class="partners-grid">
              <div class="partner-logo">
                <span class="partner-text">Logoissum</span>
              </div>
              <div class="partner-logo">
                <span class="partner-text">Logoissum</span>
              </div>
              <div class="partner-logo">
                <span class="partner-text">Logoissum</span>
              </div>
              <div class="partner-logo">
                <span class="partner-text">LOCO</span>
              </div>
              <div class="partner-logo">
                <span class="partner-text">Logoissum</span>
              </div>
              <div class="partner-logo">
                <span class="partner-text">IP/UM</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Right Column - Seat Plan & Packages -->
        <div class="event-sidebar">
          <!-- Seat Plan -->
          <div class="seat-plan-card">
            <div class="card-header">
              <h3 class="card-title h5-semibold">Seat Plan</h3>
              <div class="view-controls">
                <button
                  class="view-btn"
                  :class="{ active: showSeatPlan }"
                  @click="showSeatPlan = true"
                >
                  <i class="bi bi-diagram-3"></i>
                </button>
                <button
                  class="view-btn"
                  :class="{ active: !showSeatPlan }"
                  @click="showSeatPlan = false"
                >
                  <i class="bi bi-list"></i>
                </button>
              </div>
            </div>

            <div v-if="showSeatPlan" class="seat-plan-visual">
              <div class="seat-plan-image">
                <!-- Replace the placeholder with the actual venue SVG -->
                <img :src="VenueSvg" alt="Venue Seat Plan" class="venue-svg" />
              </div>
              <div class="seat-legend">
                <div
                  v-for="section in event.seatPlan.sections"
                  :key="section.name"
                  class="legend-item"
                >
                  <div
                    class="legend-color"
                    :style="{ backgroundColor: section.color }"
                  ></div>
                  <span class="legend-label body-12"
                    >{{ section.name }} ${{ section.price }}</span
                  >
                </div>
              </div>
            </div>

            <div class="plan-benefits">
              <div class="benefit-item">
                <span class="benefit-label body-12"
                  >Total Category Benefits</span
                >
              </div>
              <div class="benefit-item" v-if="event.vipLounge">
                <i class="bi bi-check-circle text-success"></i>
                <span class="benefit-text body-12">VIP Lounge</span>
              </div>
              <div class="benefit-item" v-if="event.backstageAccess">
                <i class="bi bi-check-circle text-success"></i>
                <span class="benefit-text body-12">Backstage Access</span>
              </div>
            </div>
          </div>

          <!-- Packages -->
          <div class="packages-card">
            <h3 class="card-title h5-semibold">Packages</h3>
            <div class="packages-list">
              <div
                v-for="pkg in event.packages"
                :key="pkg.name"
                class="package-item"
                :class="{ selected: selectedPackage?.name === pkg.name }"
                @click="selectPackage(pkg)"
              >
                <div class="package-info">
                  <h5 class="package-name body-14-semibold">{{ pkg.name }}</h5>
                  <p class="package-description body-12">
                    {{ pkg.description }}
                  </p>
                  <div class="package-features">
                    <span
                      v-for="feature in pkg.features"
                      :key="feature"
                      class="feature-tag body-10"
                    >
                      {{ feature }}
                    </span>
                  </div>
                </div>
                <div class="package-price">
                  <span class="price-value h5-semibold">${{ pkg.price }}</span>
                </div>
              </div>
            </div>

            <button
              class="btn btn-primary book-btn w-100"
              :disabled="!selectedPackage"
              @click="bookPackage"
            >
              <span class="body-14-semibold">
                {{
                  selectedPackage
                    ? `Book ${selectedPackage.name}`
                    : "Select a Package"
                }}
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Main container that works with App.vue layout */
.event-details-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: linear-gradient(
    135deg,
    var(--grey-10) 0%,
    var(--cool-grey-10) 100%
  );
  font-family: "Nunito", sans-serif;
}

/* Fixed header */
.event-header {
  background: var(--grey-10);
  border-bottom: 1px solid var(--grey-30);
  padding: 1rem 2rem;
  flex-shrink: 0;
}

.header-controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 1400px;
  margin: 0 auto;
}

.btn-back {
  background: var(--grey-20);
  border: none;
  border-radius: 12px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--secondary-100);
  transition: all 0.2s ease;
}

.btn-back:hover {
  background: var(--grey-30);
  transform: translateX(-2px);
}

.header-title {
  color: var(--secondary-100);
}

.header-actions {
  display: flex;
  gap: 0.5rem;
}

.btn-action {
  background: var(--grey-20);
  border: none;
  border-radius: 12px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--grey-60);
  transition: all 0.2s ease;
}

.btn-action:hover {
  background: var(--grey-30);
  color: var(--secondary-100);
}

/* Scrollable content area */
.event-details-content {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 2rem;
}

.loading-state,
.error-state {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
}

.loading-content,
.error-content {
  text-align: center;
  max-width: 400px;
}

.spinner-modern {
  width: 48px;
  height: 48px;
  border: 4px solid var(--grey-30);
  border-top: 4px solid var(--primary-100);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.error-icon {
  width: 80px;
  height: 80px;
  border-radius: 20px;
  background: linear-gradient(135deg, var(--grey-20), var(--grey-30));
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  color: var(--grey-60);
  font-size: 2rem;
}

.modern-btn {
  background: linear-gradient(135deg, var(--primary-100), var(--primary-40));
  border: none;
  border-radius: 12px;
  padding: 0.875rem 1.5rem;
  color: white;
  font-weight: 600;
  transition: all 0.2s ease;
}

.modern-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(242, 54, 249, 0.3);
}

.event-content {
  display: grid;
  grid-template-columns: 1fr 400px;
  gap: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.event-main {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.event-hero {
  position: relative;
  border-radius: 24px;
  overflow: hidden;
  height: 400px;
}

.event-image-container {
  position: relative;
  width: 100%;
  height: 100%;
}

.event-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.event-placeholder {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, var(--secondary-50), var(--primary-50));
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.placeholder-content {
  text-align: center;
  color: var(--secondary-100);
  opacity: 0.8;
}

.placeholder-icon {
  font-size: 3rem;
  margin-bottom: 0.5rem;
  display: block;
}

.placeholder-text {
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.event-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.8));
  padding: 2rem;
  color: white;
}

.event-title {
  margin-bottom: 1rem;
}

.event-meta {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.meta-item i {
  color: var(--primary-100);
}

.event-stats {
  display: flex;
  gap: 2rem;
}

.stat-item {
  text-align: left;
}

.stat-value {
  display: block;
  margin-bottom: 0.25rem;
}

.stat-value.price {
  color: var(--primary-100);
}

.section-card {
  background: var(--grey-10);
  border-radius: 24px;
  padding: 2rem;
  box-shadow: 0 4px 24px rgba(55, 67, 125, 0.08);
  border: 1px solid var(--grey-30);
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.5rem;
}

.section-title {
  color: var(--secondary-100);
  margin-bottom: 1.5rem;
}

.btn-expand {
  background: none;
  border: none;
  color: var(--grey-60);
  font-size: 1.25rem;
  transition: transform 0.2s ease;
}

.btn-expand:hover {
  transform: rotate(180deg);
}

.event-description {
  color: var(--grey-70);
  line-height: 1.6;
}

.terms-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.term-item {
  color: var(--grey-60);
}

.merchandise-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1.5rem;
}

.merch-item {
  text-align: center;
}

.merch-image {
  width: 100%;
  height: 120px;
  background: var(--grey-20);
  border-radius: 16px;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.merch-placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  color: var(--grey-60);
  font-size: 2rem;
}

.merch-name {
  color: var(--secondary-100);
  margin-bottom: 0.25rem;
}

.merch-price {
  color: var(--grey-60);
}

.partners-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 1rem;
  align-items: center;
}

.partner-logo {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  background: var(--grey-20);
  border-radius: 12px;
  height: 60px;
}

.partner-text {
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--grey-60);
}

.event-sidebar {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.seat-plan-card,
.packages-card {
  background: var(--grey-10);
  border-radius: 24px;
  padding: 1.5rem;
  box-shadow: 0 4px 24px rgba(55, 67, 125, 0.08);
  border: 1px solid var(--grey-30);
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.5rem;
}

.card-title {
  color: var(--secondary-100);
}

.view-controls {
  display: flex;
  background: var(--grey-20);
  border-radius: 8px;
  padding: 0.25rem;
}

.view-btn {
  background: none;
  border: none;
  padding: 0.5rem;
  border-radius: 6px;
  color: var(--grey-60);
  transition: all 0.2s ease;
}

.view-btn.active {
  background: var(--grey-10);
  color: var(--secondary-100);
  box-shadow: 0 2px 4px rgba(55, 67, 125, 0.1);
}

.seat-plan-visual {
  margin-bottom: 1.5rem;
}

/* Update the seat plan image styles */
.seat-plan-image {
  width: 100%;
  height: 600px; /* Increased height to accommodate the venue diagram */
  background: var(--grey-10);
  border-radius: 16px;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  padding: 1rem;
}

.venue-svg {
  width: 100%;
  height: 100%;
  object-fit: contain;
  filter: drop-shadow(0 2px 8px rgba(55, 67, 125, 0.1));
}

/* Remove the old placeholder styles and add hover effect */
.seat-plan-image:hover .venue-svg {
  transform: scale(1.05);
  transition: transform 0.3s ease;
}

.seat-legend {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 2px;
}

.legend-label {
  color: var(--grey-60);
}

.plan-benefits {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.benefit-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.benefit-text {
  color: var(--grey-60);
}

.text-success {
  color: #28a745;
}

.packages-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1.5rem;
  max-height: 400px;
  overflow-y: auto;
}

.package-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  border: 2px solid var(--grey-30);
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.package-item:hover {
  border-color: var(--secondary-100);
  box-shadow: 0 4px 16px rgba(55, 67, 125, 0.1);
}

.package-item.selected {
  border-color: var(--primary-100);
  background: linear-gradient(135deg, var(--primary-30), var(--grey-10));
}

.package-name {
  color: var(--secondary-100);
  margin-bottom: 0.25rem;
}

.package-description {
  color: var(--grey-60);
  margin-bottom: 0.5rem;
}

.package-features {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.feature-tag {
  background: var(--cool-grey-20);
  color: var(--secondary-100);
  padding: 0.25rem 0.5rem;
  border-radius: 6px;
  font-weight: 500;
}

.package-price {
  text-align: right;
}

.price-value {
  color: var(--primary-100);
}

.book-btn {
  background: linear-gradient(135deg, var(--primary-100), var(--primary-40));
  border: none;
  border-radius: 16px;
  padding: 1rem;
  color: white;
  transition: all 0.2s ease;
}

.book-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(242, 54, 249, 0.4);
}

.book-btn:disabled {
  background: var(--grey-40);
  cursor: not-allowed;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* Responsive Design */
@media (max-width: 1024px) {
  .event-content {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  .event-sidebar {
    order: -1;
  }
}

@media (max-width: 768px) {
  .event-details-content {
    padding: 1rem;
  }

  .event-hero {
    height: 300px;
  }

  .event-overlay {
    padding: 1.5rem;
  }

  .merchandise-grid {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  }

  .partners-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
