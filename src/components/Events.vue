<script setup>
import { ref, onMounted, computed, nextTick } from "vue";
import { useRouter } from "vue-router";

const API_BASE_URL =
  "https://eventservice01-hugcdba0e9g5ewbk.swedencentral-01.azurewebsites.net";

const router = useRouter();

// Core state
const events = ref([]);
const isLoading = ref(false);
const error = ref(null);

// Pagination state
const totalEvents = ref(0);

// UI state
const tabs = ref([
  { name: "Active", count: 0, active: true },
  { name: "Draft", count: 0, active: false },
  { name: "Past", count: 0, active: false },
]);

const searchQuery = ref("");
const selectedCategory = ref("All Category");
const selectedMonth = ref("This Month");
const viewMode = ref("grid");

// Modal state
const showCreateModal = ref(false);
const isSubmitting = ref(false);

// Form data
const newEvent = ref({
  title: "",
  description: "",
  location: "",
  eventDate: "",
  image: "",
});

// Form validation state
const formErrors = ref({
  title: "",
  description: "",
  location: "",
  eventDate: "",
});

const formTouched = ref({
  title: false,
  description: false,
  location: false,
  eventDate: false,
});

// Validation rules and functions
const validateTitle = (title) => {
  if (!title || !title.trim()) {
    return "Event title is required";
  }
  if (title.trim().length < 3) {
    return "Title must be at least 3 characters long";
  }
  if (title.trim().length > 100) {
    return "Title must be less than 100 characters";
  }
  return "";
};

const validateDescription = (description) => {
  if (!description || !description.trim()) {
    return "Event description is required";
  }
  if (description.trim().length < 10) {
    return "Description must be at least 10 characters long";
  }
  if (description.trim().length > 500) {
    return "Description must be less than 500 characters";
  }
  return "";
};

const validateLocation = (location) => {
  if (!location || !location.trim()) {
    return "Event location is required";
  }
  if (location.trim().length < 3) {
    return "Location must be at least 3 characters long";
  }
  if (location.trim().length > 100) {
    return "Location must be less than 100 characters";
  }
  return "";
};

const validateEventDate = (eventDate) => {
  if (!eventDate) {
    return "Event date and time is required";
  }

  const selectedDate = new Date(eventDate);
  const now = new Date();

  // Check if date is valid
  if (isNaN(selectedDate.getTime())) {
    return "Please enter a valid date and time";
  }

  // Check if date is in the future
  if (selectedDate <= now) {
    return "Event date must be in the future";
  }

  // Check if date is too far in the future (2 years)
  const twoYearsFromNow = new Date();
  twoYearsFromNow.setFullYear(twoYearsFromNow.getFullYear() + 2);

  if (selectedDate > twoYearsFromNow) {
    return "Event date cannot be more than 2 years in the future";
  }

  return "";
};

// Validate individual fields
const validateField = (fieldName) => {
  const value = newEvent.value[fieldName];

  switch (fieldName) {
    case "title":
      formErrors.value.title = validateTitle(value);
      break;
    case "description":
      formErrors.value.description = validateDescription(value);
      break;
    case "location":
      formErrors.value.location = validateLocation(value);
      break;
    case "eventDate":
      formErrors.value.eventDate = validateEventDate(value);
      break;
  }
};

// Validate all fields
const validateAllFields = () => {
  Object.keys(formTouched.value).forEach((fieldName) => {
    validateField(fieldName);
  });
};

// Mark field as touched and validate
const handleFieldBlur = (fieldName) => {
  formTouched.value[fieldName] = true;
  validateField(fieldName);
};

// Validate on input change if field was already touched
const handleFieldInput = (fieldName) => {
  if (formTouched.value[fieldName]) {
    validateField(fieldName);
  }
};

// Check if form is valid
const isFormValid = computed(() => {
  // Check if all required fields have values
  const hasRequiredValues =
    newEvent.value.title.trim() &&
    newEvent.value.description.trim() &&
    newEvent.value.location.trim() &&
    newEvent.value.eventDate;

  // Check if there are no validation errors
  const hasNoErrors =
    !formErrors.value.title &&
    !formErrors.value.description &&
    !formErrors.value.location &&
    !formErrors.value.eventDate;

  return hasRequiredValues && hasNoErrors;
});

// Get field validation class
const getFieldClass = (fieldName) => {
  if (!formTouched.value[fieldName]) return "";
  return formErrors.value[fieldName] ? "is-invalid" : "is-valid";
};

// API functions
const fetchEvents = async () => {
  try {
    isLoading.value = true;
    error.value = null;

    const response = await fetch(`${API_BASE_URL}/api/events`);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    console.log("API Response:", data);

    let eventsArray = [];
    if (Array.isArray(data.result)) {
      eventsArray = data.result;
    } else {
      console.warn("Unexpected API response structure:", data);
      eventsArray = [];
    }

    events.value = eventsArray.map((event) => ({
      id: event.id,
      title: event.title,
      description: event.description,
      location: event.location,
      date: formatDate(event.eventDate),
      time: formatTime(event.eventDate),
      image: event.image || "placeholder.jpg",
      category: "Event",
      ticketsLeft: "N/A",
      price: "TBD",
    }));

    totalEvents.value = events.value.length;
    updateTabCounts();
  } catch (err) {
    console.error("Error fetching events:", err);
    error.value = "Failed to load events. Please try again.";
  } finally {
    isLoading.value = false;
  }
};

const createEvent = async (eventData) => {
  try {
    const response = await fetch(`${API_BASE_URL}/api/events`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        title: eventData.title.trim(),
        description: eventData.description.trim(),
        location: eventData.location.trim(),
        eventDate: eventData.eventDate,
        image: eventData.image?.trim() || "",
      }),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    // Check if response has content before parsing JSON
    const contentType = response.headers.get("content-type");
    const text = await response.text();

    if (!text || text.trim() === "") {
      // If response is empty, return a basic object
      console.log(
        "Empty response from server, event likely created successfully"
      );
      return {
        id: Date.now().toString(),
        title: eventData.title.trim(),
        description: eventData.description.trim(),
        location: eventData.location.trim(),
        eventDate: eventData.eventDate,
        image: eventData.image?.trim() || null,
      };
    }

    try {
      const createdEvent = JSON.parse(text);
      return createdEvent;
    } catch (jsonError) {
      console.warn("Invalid JSON response:", text);
      return {
        id: Date.now().toString(),
        title: eventData.title.trim(),
        description: eventData.description.trim(),
        location: eventData.location.trim(),
        eventDate: eventData.eventDate,
        image: eventData.image?.trim() || null,
      };
    }
  } catch (err) {
    console.error("Error creating event:", err);
    throw err;
  }
};

// Helper functions
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

const updateTabCounts = () => {
  const now = new Date();
  tabs.value[0].count = events.value.filter(
    (e) => new Date(e.date) >= now
  ).length;
  tabs.value[1].count = 0; // Draft events - implement when backend supports it
  tabs.value[2].count = events.value.filter(
    (e) => new Date(e.date) < now
  ).length;
};

// Tab switching functionality
const switchTab = (selectedTab) => {
  tabs.value.forEach((tab) => {
    tab.active = tab.name === selectedTab.name;
  });
};

// Image error handling
const handleImageError = (event) => {
  event.target.style.display = "none";
  const placeholder = event.target.nextElementSibling;
  if (placeholder) {
    placeholder.style.display = "flex";
  }
};

const openCreateModal = () => {
  showCreateModal.value = true;
  // Prevent body scroll when modal is open
  document.body.classList.add("modal-open");
  resetForm();
};

const closeCreateModal = () => {
  showCreateModal.value = false;
  // Restore body scroll when modal is closed
  document.body.classList.remove("modal-open");
  resetForm();
};

const resetForm = () => {
  // Reset form data
  newEvent.value = {
    title: "",
    description: "",
    location: "",
    eventDate: "",
    image: "",
  };

  // Reset validation state
  formErrors.value = {
    title: "",
    description: "",
    location: "",
    eventDate: "",
  };

  formTouched.value = {
    title: false,
    description: false,
    location: false,
    eventDate: false,
  };
};

const handleCreateEvent = async () => {
  try {
    // Mark all fields as touched to show validation errors
    Object.keys(formTouched.value).forEach((key) => {
      formTouched.value[key] = true;
    });

    // Validate all fields
    validateAllFields();

    // Check if form is valid
    if (!isFormValid.value) {
      console.log("Form validation failed");
      return;
    }

    isSubmitting.value = true;

    // Create the event
    const createdEvent = await createEvent(newEvent.value);

    // Refresh events list to get latest data
    await fetchEvents();

    // Close modal on success
    closeCreateModal();

    console.log("Event created successfully:", createdEvent);
  } catch (error) {
    console.error("Error creating event:", error);

    // Show user-friendly error message
    const errorMessage = error.message?.includes("HTTP error")
      ? "Failed to create event due to server error. Please try again."
      : "Failed to create event. Please check your connection and try again.";

    alert(errorMessage);
  } finally {
    isSubmitting.value = false;
  }
};

// Add slug generation function
const generateSlug = (title) => {
  return title
    .toLowerCase()
    .replace(/[^\w\s-]/g, "") // Remove special characters
    .replace(/\s+/g, "-") // Replace spaces with hyphens
    .replace(/-+/g, "-") // Replace multiple hyphens with single
    .trim("-"); // Remove leading/trailing hyphens
};

const viewEventDetails = (event) => {
  // Generate slug from event title, fallback to ID if no title
  const slug = event.title ? generateSlug(event.title) : `event-${event.id}`;
  router.push(`/events/${slug}`);
};

const adjustForFooter = () => {
  const footer = document.querySelector("footer");
  if (footer) {
    const footerHeight = footer.offsetHeight;
    const eventsPage = document.querySelector(".events-page");
    if (eventsPage) {
      eventsPage.style.paddingBottom = `${footerHeight + 40}px`; // Extra 40px for breathing room
    }
  }
};

// Initialize component
onMounted(() => {
  document.body.classList.remove("modal-open");
  fetchEvents();

  // Adjust for footer after component is mounted
  nextTick(() => {
    adjustForFooter();
  });
});
</script>

<template>
  <div class="events-page">
    <!-- Loading State -->
    <div v-if="isLoading" class="loading-state">
      <div class="loading-content">
        <div class="spinner-modern"></div>
        <h5 class="h5-semibold mt-3">Loading Events</h5>
        <p class="body-14 text-muted">Fetching the latest events for you...</p>
      </div>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="error-state">
      <div class="error-content">
        <div class="error-icon">
          <i class="bi bi-exclamation-triangle"></i>
        </div>
        <h5 class="h5-semibold mt-3">Something went wrong</h5>
        <p class="body-14 text-muted mb-4">{{ error }}</p>
        <button class="btn btn-primary modern-btn" @click="fetchEvents">
          <i class="bi bi-arrow-clockwise me-2"></i>
          Try Again
        </button>
      </div>
    </div>

    <!-- Main Content -->
    <div v-else class="events-content">
      <!-- Page Header -->
      <div class="page-header">
        <div class="header-content">
          <div class="header-text">
            <h1 class="h1-semibold page-title">Events</h1>
            <p class="body-16 text-muted">Discover and manage amazing events</p>
          </div>
          <button
            class="btn btn-primary modern-btn-primary"
            @click="openCreateModal"
          >
            <i class="bi bi-plus-circle me-2"></i>
            <span class="body-14-semibold">Create Event</span>
          </button>
        </div>
      </div>

      <!-- Controls Section -->
      <div class="controls-section">
        <!-- Tabs -->
        <div class="tabs-container">
          <div class="modern-tabs">
            <button
              v-for="tab in tabs"
              :key="tab.name"
              class="tab-button"
              :class="{ active: tab.active }"
              @click="switchTab(tab)"
            >
              <span class="tab-label body-14-semibold">{{ tab.name }}</span>
              <span class="tab-count">{{ tab.count }}</span>
            </button>
          </div>
        </div>

        <!-- Search and Filters -->
        <div class="filters-section">
          <div class="search-container">
            <div class="modern-search">
              <i class="bi bi-search search-icon"></i>
              <input
                type="text"
                class="search-input"
                placeholder="Search events, locations..."
                v-model="searchQuery"
              />
            </div>
          </div>

          <div class="filter-controls">
            <!-- Category Filter -->
            <div class="dropdown filter-dropdown">
              <button
                class="btn modern-filter-btn dropdown-toggle"
                type="button"
                data-bs-toggle="dropdown"
              >
                <i class="bi bi-funnel me-2"></i>
                <span class="body-14">{{ selectedCategory }}</span>
              </button>
              <ul class="dropdown-menu modern-dropdown">
                <li>
                  <a
                    class="dropdown-item body-14"
                    href="#"
                    @click.prevent="selectedCategory = 'All Category'"
                  >
                    <i class="bi bi-grid me-2"></i>
                    All Category
                  </a>
                </li>
                <li>
                  <a
                    class="dropdown-item body-14"
                    href="#"
                    @click.prevent="selectedCategory = 'Music'"
                  >
                    <i class="bi bi-music-note me-2"></i>
                    Music
                  </a>
                </li>
                <li>
                  <a
                    class="dropdown-item body-14"
                    href="#"
                    @click.prevent="selectedCategory = 'Food & Culinary'"
                  >
                    <i class="bi bi-cup-hot me-2"></i>
                    Food & Culinary
                  </a>
                </li>
              </ul>
            </div>

            <!-- Time Filter -->
            <div class="dropdown filter-dropdown">
              <button
                class="btn modern-filter-btn dropdown-toggle"
                type="button"
                data-bs-toggle="dropdown"
              >
                <i class="bi bi-calendar me-2"></i>
                <span class="body-14">{{ selectedMonth }}</span>
              </button>
              <ul class="dropdown-menu modern-dropdown">
                <li>
                  <a
                    class="dropdown-item body-14"
                    href="#"
                    @click.prevent="selectedMonth = 'This Month'"
                  >
                    <i class="bi bi-calendar-event me-2"></i>
                    This Month
                  </a>
                </li>
                <li>
                  <a
                    class="dropdown-item body-14"
                    href="#"
                    @click.prevent="selectedMonth = 'Next Month'"
                  >
                    <i class="bi bi-calendar-plus me-2"></i>
                    Next Month
                  </a>
                </li>
                <li>
                  <a
                    class="dropdown-item body-14"
                    href="#"
                    @click.prevent="selectedMonth = 'All Time'"
                  >
                    <i class="bi bi-calendar2 me-2"></i>
                    All Time
                  </a>
                </li>
              </ul>
            </div>

            <!-- View Toggle -->
            <div class="view-toggle">
              <button
                class="btn view-btn"
                :class="{ active: viewMode === 'grid' }"
                @click="viewMode = 'grid'"
              >
                <i class="bi bi-grid-3x3-gap"></i>
              </button>
              <button
                class="btn view-btn"
                :class="{ active: viewMode === 'list' }"
                @click="viewMode = 'list'"
              >
                <i class="bi bi-list"></i>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Events List -->
      <div class="events-container">
        <!-- Empty State -->
        <div v-if="events.length === 0" class="empty-state">
          <div class="empty-content">
            <div class="empty-icon">
              <i class="bi bi-calendar-x"></i>
            </div>
            <h4 class="h4-semibold mt-3">No events found</h4>
            <p class="body-14 text-muted mb-4">
              Create your first event to get started!
            </p>
            <button class="btn btn-primary modern-btn" @click="openCreateModal">
              <i class="bi bi-plus-circle me-2"></i>
              Create Your First Event
            </button>
          </div>
        </div>

        <!-- Events Grid -->
        <div v-else class="events-grid">
          <div
            v-for="event in events"
            :key="event.id"
            class="event-card-modern"
            @click="viewEventDetails(event)"
            style="cursor: pointer"
          >
            <!-- Event Image -->
            <div class="event-image-container">
              <img
                v-if="event.image && event.image !== 'placeholder.jpg'"
                :src="event.image"
                :alt="event.title"
                class="event-image"
                @error="handleImageError"
              />
              <div v-else class="event-placeholder">
                <div class="placeholder-content">
                  <i class="bi bi-calendar-event placeholder-icon"></i>
                  <span class="placeholder-text">Event</span>
                </div>
              </div>

              <!-- Event Category Badge -->
              <div class="category-badge-overlay">
                <span class="category-badge body-10-semibold">{{
                  event.category
                }}</span>
              </div>
            </div>

            <!-- Event Content -->
            <div class="event-content">
              <div class="event-header">
                <h5 class="event-title h5-semibold">{{ event.title }}</h5>
                <p class="event-description body-14">{{ event.description }}</p>
              </div>

              <div class="event-details">
                <div class="detail-item">
                  <i class="bi bi-geo-alt detail-icon"></i>
                  <span class="detail-text body-12">{{ event.location }}</span>
                </div>
                <div class="detail-item">
                  <i class="bi bi-calendar detail-icon"></i>
                  <span class="detail-text body-12"
                    >{{ event.date }} • {{ event.time }}</span
                  >
                </div>
              </div>

              <div class="event-footer">
                <div class="event-stats">
                  <div class="stat-item">
                    <span class="stat-value body-14-semibold">{{
                      event.ticketsLeft
                    }}</span>
                    <span class="stat-label body-10">Tickets Left</span>
                  </div>
                  <div class="stat-item">
                    <span class="stat-value price title-16-semibold">{{
                      event.price
                    }}</span>
                    <span class="stat-label body-10">Price</span>
                  </div>
                </div>

                <div class="event-actions">
                  <button
                    class="btn action-btn-secondary"
                    @click="viewEventDetails(event)"
                  >
                    <i class="bi bi-eye"></i>
                  </button>
                  <button
                    class="btn action-btn-primary"
                    @click="viewEventDetails(event)"
                  >
                    <i class="bi bi-ticket-perforated me-1"></i>
                    <span class="body-12-semibold">Book</span>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Pagination -->
      <div class="pagination-section" v-if="events.length > 0">
        <div class="pagination-info">
          <span class="body-12 text-muted">
            Showing <strong>{{ events.length }}</strong> out of
            <strong>{{ totalEvents }}</strong> events
          </span>
        </div>

        <nav class="modern-pagination">
          <ul class="pagination-list">
            <li class="page-item">
              <button class="page-btn" aria-label="Previous">
                <i class="bi bi-chevron-left"></i>
              </button>
            </li>
            <li class="page-item active">
              <button class="page-btn body-14-semibold">1</button>
            </li>
            <li class="page-item">
              <button class="page-btn body-14">2</button>
            </li>
            <li class="page-item">
              <button class="page-btn body-14">3</button>
            </li>
            <li class="page-item disabled">
              <span class="page-btn body-14">...</span>
            </li>
            <li class="page-item">
              <button class="page-btn body-14">8</button>
            </li>
            <li class="page-item">
              <button class="page-btn" aria-label="Next">
                <i class="bi bi-chevron-right"></i>
              </button>
            </li>
          </ul>
        </nav>
      </div>
    </div>

    <!-- Create Event Modal -->
    <div
      class="modal fade custom-modal"
      :class="{ show: showCreateModal }"
      :style="{ display: showCreateModal ? 'block' : 'none' }"
      tabindex="-1"
      @click.self="closeCreateModal"
    >
      <div class="modal-dialog modal-lg modal-dialog-centered">
        <div class="modal-content custom-modal-content">
          <div class="modal-header custom-modal-header">
            <div class="d-flex align-items-center">
              <div class="modal-icon me-3">
                <i class="bi bi-calendar-plus"></i>
              </div>
              <div>
                <h4 class="h4-semibold mb-1">Create New Event</h4>
                <p class="body-12 text-muted mb-0">
                  Fill in the details to create your event
                </p>
              </div>
            </div>
            <button
              type="button"
              class="btn-close custom-btn-close"
              @click="closeCreateModal"
              aria-label="Close"
            >
              <i class="bi bi-x-lg"></i>
            </button>
          </div>

          <div class="modal-body custom-modal-body">
            <form @submit.prevent="handleCreateEvent">
              <div class="row g-4">
                <!-- Event Title -->
                <div class="col-12">
                  <div class="form-group">
                    <label
                      for="eventTitle"
                      class="form-label body-14-semibold mb-2"
                    >
                      <i class="bi bi-type me-2 text-muted"></i>Event Title
                      <span class="text-danger">*</span>
                    </label>
                    <input
                      type="text"
                      class="form-control custom-form-control"
                      :class="getFieldClass('title')"
                      id="eventTitle"
                      placeholder="Enter an engaging event title"
                      v-model="newEvent.title"
                      @blur="handleFieldBlur('title')"
                      @input="handleFieldInput('title')"
                      :maxlength="100"
                      required
                    />
                    <div
                      class="d-flex justify-content-between align-items-start mt-1"
                    >
                      <div
                        class="invalid-feedback d-block"
                        v-if="formTouched.title && formErrors.title"
                      >
                        <i class="bi bi-exclamation-circle me-1"></i>
                        {{ formErrors.title }}
                      </div>
                      <div
                        class="valid-feedback d-block"
                        v-else-if="
                          formTouched.title &&
                          !formErrors.title &&
                          newEvent.title.trim()
                        "
                      >
                        <i class="bi bi-check-circle me-1"></i>
                        Looks good!
                      </div>
                      <small class="text-muted ms-auto body-10">
                        {{ newEvent.title.length }}/100
                      </small>
                    </div>
                  </div>
                </div>

                <!-- Event Description -->
                <div class="col-12">
                  <div class="form-group">
                    <label
                      for="eventDescription"
                      class="form-label body-14-semibold mb-2"
                    >
                      <i class="bi bi-text-paragraph me-2 text-muted"></i
                      >Description
                      <span class="text-danger">*</span>
                    </label>
                    <textarea
                      class="form-control custom-form-control"
                      :class="getFieldClass('description')"
                      id="eventDescription"
                      rows="4"
                      placeholder="Describe your event in detail..."
                      v-model="newEvent.description"
                      @blur="handleFieldBlur('description')"
                      @input="handleFieldInput('description')"
                      :maxlength="500"
                      required
                    ></textarea>
                    <div
                      class="d-flex justify-content-between align-items-start mt-1"
                    >
                      <div
                        class="invalid-feedback d-block"
                        v-if="formTouched.description && formErrors.description"
                      >
                        <i class="bi bi-exclamation-circle me-1"></i>
                        {{ formErrors.description }}
                      </div>
                      <div
                        class="valid-feedback d-block"
                        v-else-if="
                          formTouched.description &&
                          !formErrors.description &&
                          newEvent.description.trim()
                        "
                      >
                        <i class="bi bi-check-circle me-1"></i>
                        Great description!
                      </div>
                      <small class="text-muted ms-auto body-10">
                        {{ newEvent.description.length }}/500
                      </small>
                    </div>
                    <div
                      class="form-text body-12"
                      v-if="!formTouched.description || !formErrors.description"
                    >
                      Tell people what makes your event special
                    </div>
                  </div>
                </div>

                <!-- Location and Date Row -->
                <div class="col-md-6">
                  <div class="form-group">
                    <label
                      for="eventLocation"
                      class="form-label body-14-semibold mb-2"
                    >
                      <i class="bi bi-geo-alt me-2 text-muted"></i>Location
                      <span class="text-danger">*</span>
                    </label>
                    <div class="input-group custom-input-group">
                      <span
                        class="input-group-text"
                        :class="getFieldClass('location')"
                      >
                        <i class="bi bi-building"></i>
                      </span>
                      <input
                        type="text"
                        class="form-control custom-form-control"
                        :class="getFieldClass('location')"
                        id="eventLocation"
                        placeholder="Venue name or address"
                        v-model="newEvent.location"
                        @blur="handleFieldBlur('location')"
                        @input="handleFieldInput('location')"
                        required
                      />
                    </div>
                    <div
                      class="invalid-feedback d-block"
                      v-if="formTouched.location && formErrors.location"
                    >
                      <i class="bi bi-exclamation-circle me-1"></i>
                      {{ formErrors.location }}
                    </div>
                    <div
                      class="valid-feedback d-block"
                      v-else-if="
                        formTouched.location &&
                        !formErrors.location &&
                        newEvent.location.trim()
                      "
                    >
                      <i class="bi bi-check-circle me-1"></i>
                      Location looks good!
                    </div>
                  </div>
                </div>

                <div class="col-md-6">
                  <div class="form-group">
                    <label
                      for="eventDate"
                      class="form-label body-14-semibold mb-2"
                    >
                      <i class="bi bi-calendar-event me-2 text-muted"></i>Date &
                      Time
                      <span class="text-danger">*</span>
                    </label>
                    <input
                      type="datetime-local"
                      class="form-control custom-form-control"
                      :class="getFieldClass('eventDate')"
                      id="eventDate"
                      v-model="newEvent.eventDate"
                      @blur="handleFieldBlur('eventDate')"
                      @input="handleFieldInput('eventDate')"
                      required
                    />
                    <div
                      class="invalid-feedback d-block"
                      v-if="formTouched.eventDate && formErrors.eventDate"
                    >
                      <i class="bi bi-exclamation-circle me-1"></i>
                      {{ formErrors.eventDate }}
                    </div>
                    <div
                      class="valid-feedback d-block"
                      v-else-if="
                        formTouched.eventDate &&
                        !formErrors.eventDate &&
                        newEvent.eventDate
                      "
                    >
                      <i class="bi bi-check-circle me-1"></i>
                      Perfect timing!
                    </div>
                  </div>
                </div>

                <!-- Event Image -->
                <div class="col-12">
                  <div class="form-group">
                    <label
                      for="eventImage"
                      class="form-label body-14-semibold mb-2"
                    >
                      <i class="bi bi-image me-2 text-muted"></i>Event Image
                      <span class="badge bg-light ms-2 body-10">Optional</span>
                    </label>
                    <div class="image-upload-container">
                      <input
                        type="url"
                        class="form-control custom-form-control"
                        id="eventImage"
                        placeholder="https://example.com/your-event-image.jpg"
                        v-model="newEvent.image"
                      />
                      <div class="form-text body-12 d-flex align-items-center">
                        <i class="bi bi-info-circle me-1"></i>
                        Add an image URL to make your event more attractive
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </form>
          </div>

          <div class="modal-footer custom-modal-footer">
            <div class="d-flex w-100 gap-3">
              <button
                type="button"
                class="btn btn-outline-secondary custom-btn-secondary flex-fill"
                @click="closeCreateModal"
                :disabled="isSubmitting"
              >
                <i class="bi bi-x-circle me-2"></i>
                <span class="body-14-semibold">Cancel</span>
              </button>
              <button
                type="button"
                class="btn btn-primary custom-btn-primary flex-fill"
                @click="handleCreateEvent"
                :disabled="isSubmitting || !isFormValid"
              >
                <span
                  v-if="isSubmitting"
                  class="spinner-border spinner-border-sm me-2"
                  role="status"
                ></span>
                <i v-else class="bi bi-plus-circle me-2"></i>
                <span class="body-14-semibold">{{
                  isSubmitting ? "Creating Event..." : "Create Event"
                }}</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div
      class="modal-backdrop custom-modal-backdrop"
      :class="{ show: showCreateModal }"
      v-if="showCreateModal"
    ></div>
  </div>
</template>

<style scoped>
/* Page Layout - Fix scrolling issue */
.events-page {
  min-height: 100vh;
  background: linear-gradient(
    135deg,
    var(--grey-10) 0%,
    var(--cool-grey-10) 100%
  );
  font-family: "Nunito", sans-serif;
  /* Remove any overflow hidden that might prevent scrolling */
  overflow-x: hidden;
  overflow-y: auto;
  /* Add padding bottom to prevent footer overlap */
  padding-bottom: 200px; /* Increase this value to ensure footer clearance */
}

.events-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  /* Ensure content can scroll */
  position: relative;
  z-index: 1;
  /* Add bottom margin for extra footer clearance */
  margin-bottom: 5rem;
}

/* Page Header */
.page-header {
  margin-bottom: 2rem;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 2rem 0;
  border-bottom: 1px solid var(--grey-30);
}

.page-title {
  font-weight: 800;
  color: var(--secondary-100);
  margin-bottom: 0.5rem;
  background: linear-gradient(135deg, var(--secondary-100), var(--primary-100));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.modern-btn-primary {
  background: linear-gradient(135deg, var(--primary-100), var(--primary-40));
  border: none;
  border-radius: 16px;
  padding: 1rem 2rem;
  color: white;
  font-weight: 600;
  box-shadow: 0 8px 32px rgba(242, 54, 249, 0.3);
  transition: all 0.3s ease;
}

.modern-btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 40px rgba(242, 54, 249, 0.4);
}

/* Controls Section */
.controls-section {
  background: var(--grey-10);
  border-radius: 24px;
  padding: 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 24px rgba(55, 67, 125, 0.08);
  border: 1px solid var(--grey-30);
}

/* Modern Tabs */
.tabs-container {
  margin-bottom: 2rem;
}

.modern-tabs {
  display: flex;
  background: var(--grey-20);
  border-radius: 16px;
  padding: 0.5rem;
  width: fit-content;
}

.tab-button {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.875rem 1.5rem;
  border: none;
  background: transparent;
  border-radius: 12px;
  font-weight: 600;
  color: var(--grey-60);
  transition: all 0.2s ease;
  cursor: pointer;
}

.tab-button.active {
  background: var(--grey-10);
  color: var(--secondary-100);
  box-shadow: 0 2px 8px rgba(55, 67, 125, 0.15);
}

.tab-count {
  background: var(--cool-grey-20);
  color: var(--secondary-100);
  padding: 0.25rem 0.5rem;
  border-radius: 8px;
  font-size: 0.75rem;
  font-weight: 700;
  min-width: 24px;
  text-align: center;
}

.tab-button.active .tab-count {
  background: var(--primary-30);
  color: var(--primary-100);
}

/* Filters Section */
.filters-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.search-container {
  flex: 1;
  max-width: 400px;
}

.modern-search {
  position: relative;
  width: 100%;
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--grey-50);
  font-size: 1rem;
}

.search-input {
  width: 100%;
  padding: 1rem 1rem 1rem 3rem;
  border: 2px solid var(--grey-30);
  border-radius: 16px;
  background: var(--grey-10);
  font-size: 0.875rem;
  font-family: "Nunito", sans-serif;
  transition: all 0.2s ease;
}

.search-input:focus {
  outline: none;
  border-color: var(--secondary-100);
  box-shadow: 0 0 0 4px rgba(55, 67, 125, 0.1);
}

.filter-controls {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.modern-filter-btn {
  background: var(--grey-10);
  border: 2px solid var(--grey-30);
  border-radius: 12px;
  padding: 0.875rem 1.25rem;
  color: var(--grey-70);
  font-weight: 600;
  transition: all 0.2s ease;
}

.modern-filter-btn:hover {
  border-color: var(--secondary-100);
  color: var(--secondary-100);
}

.modern-dropdown {
  border: none;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(55, 67, 125, 0.15);
  padding: 0.5rem;
  margin-top: 0.5rem;
}

.modern-dropdown .dropdown-item {
  border-radius: 12px;
  padding: 0.75rem 1rem;
  margin-bottom: 0.25rem;
  font-weight: 500;
  color: var(--grey-70);
  transition: all 0.2s ease;
}

.modern-dropdown .dropdown-item:hover {
  background: var(--cool-grey-10);
  color: var(--secondary-100);
}

/* View Toggle */
.view-toggle {
  display: flex;
  background: var(--grey-20);
  border-radius: 12px;
  padding: 0.25rem;
}

.view-btn {
  padding: 0.75rem;
  border: none;
  background: transparent;
  border-radius: 8px;
  color: var(--grey-60);
  transition: all 0.2s ease;
}

.view-btn.active {
  background: var(--grey-10);
  color: var(--secondary-100);
  box-shadow: 0 2px 8px rgba(55, 67, 125, 0.1);
}

/* Events Grid - Fix potential layout issues */
.events-container {
  margin-bottom: 2rem;
}

.events-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(380px, 1fr));
  gap: 2rem;
  width: 100%;
  position: relative;
  margin-bottom: 4rem;
}

/* Fix any potential overflow issues with event cards */
.event-card-modern {
  background: var(--grey-10);
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 4px 24px rgba(55, 67, 125, 0.08);
  border: 1px solid var(--grey-30);
  transition: all 0.3s ease;
  cursor: pointer;
  /* Ensure cards don't cause overflow */
  max-width: 100%;
  position: relative;
}

.event-card-modern:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 40px rgba(55, 67, 125, 0.15);
}

/* Event Image */
.event-image-container {
  position: relative;
  height: 200px;
  overflow: hidden;
}

.event-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.event-card-modern:hover .event-image {
  transform: scale(1.05);
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

.event-placeholder::before {
  content: "";
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.2),
    transparent
  );
  animation: shimmer 2s infinite;
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

.category-badge-overlay {
  position: absolute;
  top: 1rem;
  left: 1rem;
}

.category-badge {
  background: rgba(255, 255, 255, 0.95);
  color: var(--secondary-100);
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 600;
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Event Content */
.event-content {
  padding: 1.5rem;
}

.event-header {
  margin-bottom: 1.5rem;
}

.event-title {
  color: var(--secondary-100);
  font-weight: 700;
  margin-bottom: 0.5rem;
  line-height: 1.3;
}

.event-description {
  color: var(--grey-60);
  font-size: 0.875rem;
  line-height: 1.5;
  margin: 0;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Event Details */
.event-details {
  margin-bottom: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.detail-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.detail-icon {
  color: var(--secondary-100);
  font-size: 1rem;
  width: 16px;
  text-align: center;
}

.detail-text {
  color: var(--grey-70);
  font-size: 0.875rem;
  font-weight: 500;
}

/* Event Footer */
.event-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid var(--grey-30);
}

.event-stats {
  display: flex;
  gap: 1.5rem;
}

.stat-item {
  text-align: center;
}

.stat-value {
  display: block;
  font-weight: 700;
  color: var(--grey-90);
  font-size: 0.875rem;
}

.stat-value.price {
  color: var(--primary-100);
  font-size: 1rem;
}

.stat-label {
  display: block;
  font-size: 0.75rem;
  color: var(--grey-50);
  margin-top: 0.25rem;
}

.event-actions {
  display: flex;
  gap: 0.5rem;
}

.action-btn-secondary {
  padding: 0.75rem;
  border: 2px solid var(--grey-30);
  background: var(--grey-10);
  color: var(--grey-60);
  border-radius: 12px;
  transition: all 0.2s ease;
}

.action-btn-secondary:hover {
  border-color: var(--secondary-100);
  color: var(--secondary-100);
}

.action-btn-primary {
  background: linear-gradient(135deg, var(--primary-100), var(--primary-40));
  border: none;
  color: white;
  padding: 0.75rem 1.25rem;
  border-radius: 12px;
  font-weight: 600;
  font-size: 0.875rem;
  transition: all 0.2s ease;
}

.action-btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(242, 54, 249, 0.3);
}

/* States */
.loading-state,
.error-state,
.empty-state {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
}

.loading-content,
.error-content,
.empty-content {
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

.error-icon,
.empty-icon {
  width: 80px;
  height: 80px;
  border-radius: 20px;
  background: linear-gradient(135deg, var(--yellow-90), var(--yellow-100));
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  color: var(--yellow-100);
  font-size: 2rem;
}

.empty-icon {
  background: linear-gradient(135deg, var(--cool-grey-20), var(--cool-grey-30));
  color: var(--grey-60);
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

/* Pagination */
.pagination-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 2rem 0;
  border-top: 1px solid var(--grey-30);
}

.modern-pagination {
  display: flex;
}

.pagination-list {
  display: flex;
  list-style: none;
  gap: 0.5rem;
  margin: 0;
  padding: 0;
}

.page-btn {
  padding: 0.75rem 1rem;
  border: 2px solid var(--grey-30);
  background: var(--grey-10);
  color: var(--grey-60);
  border-radius: 12px;
  font-weight: 600;
  transition: all 0.2s ease;
  cursor: pointer;
}

.page-item.active .page-btn {
  background: var(--primary-100);
  border-color: var(--primary-100);
  color: white;
}

.page-btn:hover:not(.page-item.disabled .page-btn) {
  border-color: var(--secondary-100);
  color: var(--secondary-100);
}

/* Modal Styles - Fix backdrop preventing scroll */
.custom-modal {
  font-family: "Nunito", sans-serif;
  /* Ensure modal doesn't prevent page scroll when closed */
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1050;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  overflow-y: auto;
}

.custom-modal-backdrop {
  background: linear-gradient(
    135deg,
    rgba(55, 67, 125, 0.8),
    rgba(242, 54, 249, 0.3)
  );
  backdrop-filter: blur(8px);
  transition: all 0.3s ease;
  /* Fix backdrop preventing scroll */
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1040;
  width: 100vw;
  height: 100vh;
}

.custom-modal-backdrop.show {
  opacity: 1;
}

.custom-modal-content {
  border: none;
  border-radius: 24px;
  box-shadow: 0 24px 64px rgba(55, 67, 125, 0.15),
    0 8px 32px rgba(242, 54, 249, 0.1);
  background: var(--grey-10);
  overflow: hidden;
  transform: scale(0.9);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.custom-modal.show .custom-modal-content {
  transform: scale(1);
}

.custom-modal-header {
  background: linear-gradient(
    135deg,
    var(--grey-10) 0%,
    var(--cool-grey-10) 100%
  );
  border-bottom: 1px solid var(--grey-30);
  padding: 2rem 2rem 1.5rem;
  position: relative;
}

.custom-modal-header::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, var(--secondary-100), var(--primary-100));
}

.modal-icon {
  width: 48px;
  height: 48px;
  background: linear-gradient(
    135deg,
    var(--secondary-100),
    var(--secondary-90)
  );
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--grey-10);
  font-size: 1.25rem;
  box-shadow: 0 4px 16px rgba(55, 67, 125, 0.2);
}

.custom-btn-close {
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
  font-size: 0.875rem;
}

.custom-btn-close:hover {
  background: var(--grey-30);
  color: var(--secondary-100);
  transform: scale(1.05);
}

.custom-modal-body {
  padding: 2rem;
  background: var(--grey-10);
}

.form-group {
  margin-bottom: 0;
}

.form-label {
  color: var(--grey-90);
  font-weight: 600;
  display: flex;
  align-items: center;
}

.form-label i {
  color: var(--secondary-100);
}

.custom-form-control {
  border: 2px solid var(--grey-30);
  border-radius: 12px;
  padding: 0.875rem 1rem;
  font-size: 0.875rem;
  background: var(--grey-10);
  transition: all 0.2s ease;
  font-family: "Nunito", sans-serif;
}

.custom-form-control:focus {
  border-color: var(--secondary-100);
  box-shadow: 0 0 0 4px rgba(55, 67, 125, 0.1);
  background: var(--grey-10);
}

.custom-form-control::placeholder {
  color: var(--grey-50);
  font-style: italic;
}

.custom-input-group .input-group-text {
  background: var(--grey-20);
  border: 2px solid var(--grey-30);
  border-right: none;
  border-radius: 12px 0 0 12px;
  color: var(--secondary-100);
  padding: 0.875rem 1rem;
}

.custom-input-group .custom-form-control {
  border-left: none;
  border-radius: 0 12px 12px 0;
}

.custom-input-group .custom-form-control:focus {
  border-left: none;
}

.image-upload-container {
  position: relative;
}

.form-text {
  color: var(--grey-60);
  font-size: 0.75rem;
  margin-top: 0.5rem;
}

.badge {
  font-size: 0.625rem;
  padding: 0.25rem 0.5rem;
  border-radius: 6px;
  font-weight: 500;
}

.custom-modal-footer {
  background: var(--grey-20);
  border-top: 1px solid var(--grey-30);
  padding: 1.5rem 2rem;
}

.custom-btn-primary {
  background: linear-gradient(135deg, var(--primary-100), var(--primary-40));
  border: none;
  border-radius: 12px;
  padding: 0.875rem 1.5rem;
  font-weight: 600;
  font-size: 0.875rem;
  color: var(--grey-10);
  transition: all 0.2s ease;
  box-shadow: 0 4px 16px rgba(242, 54, 249, 0.3);
  position: relative;
  overflow: hidden;
}

.custom-btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(242, 54, 249, 0.4);
}

.custom-btn-primary:disabled {
  opacity: 0.7;
  transform: none;
  box-shadow: 0 4px 16px rgba(242, 54, 249, 0.2);
  cursor: not-allowed;
}

.custom-btn-secondary {
  background: var(--grey-10);
  border: 2px solid var(--grey-40);
  border-radius: 12px;
  padding: 0.875rem 1.5rem;
  font-weight: 600;
  font-size: 0.875rem;
  color: var(--grey-60);
  transition: all 0.2s ease;
}

.custom-btn-secondary:hover:not(:disabled) {
  background: var(--grey-20);
  border-color: var(--secondary-100);
  color: var(--secondary-100);
  transform: translateY(-1px);
}

/* Form Validation Styles */
.custom-form-control.is-invalid {
  border-color: #dc3545;
  box-shadow: 0 0 0 4px rgba(220, 53, 69, 0.1);
}

.custom-form-control.is-valid {
  border-color: #28a745;
  box-shadow: 0 0 0 4px rgba(40, 167, 69, 0.1);
}

.custom-input-group .input-group-text.is-invalid {
  border-color: #dc3545;
}

.custom-input-group .input-group-text.is-valid {
  border-color: #28a745;
}

.invalid-feedback {
  color: #dc3545;
  font-size: 0.75rem;
  margin-top: 0.25rem;
  display: flex;
  align-items: center;
}

.valid-feedback {
  color: #28a745;
  font-size: 0.75rem;
  margin-top: 0.25rem;
  display: flex;
  align-items: center;
}

.text-danger {
  color: #dc3545 !important;
}

/* Character count styling */
.form-group small {
  font-size: 0.6875rem;
  color: var(--grey-50);
}

/* Required field indicator */
.form-label .text-danger {
  margin-left: 2px;
}

/* Validation feedback animations */
.invalid-feedback,
.valid-feedback {
  animation: fadeInUp 0.3s ease;
}

/* Animations */
@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@keyframes shimmer {
  0% {
    left: -100%;
  }
  100% {
    left: 100%;
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.form-group {
  animation: slideInUp 0.3s ease forwards;
}

.form-group:nth-child(1) {
  animation-delay: 0.1s;
}
.form-group:nth-child(2) {
  animation-delay: 0.2s;
}
.form-group:nth-child(3) {
  animation-delay: 0.3s;
}

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsive Design - Fix mobile scrolling */
@media (max-width: 1200px) {
  .events-grid {
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  }
}

@media (max-width: 768px) {
  .events-content {
    padding: 1rem;
  }

  .header-content {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  .filters-section {
    flex-direction: column;
    align-items: stretch;
  }

  .filter-controls {
    justify-content: center;
    flex-wrap: wrap;
  }

  .events-grid {
    grid-template-columns: 1fr;
    /* Ensure mobile grid doesn't cause issues */
    width: 100%;
  }

  .pagination-section {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  /* Fix mobile viewport issues */
  .events-page {
    min-height: 100vh;
    height: auto;
  }

  /* Modal adjustments for mobile */
  .custom-modal-header {
    padding: 1.5rem 1.5rem 1rem;
  }

  .custom-modal-body {
    padding: 1.5rem;
  }

  .custom-modal-footer {
    padding: 1rem 1.5rem;
  }

  .modal-dialog {
    margin: 1rem;
  }

  .d-flex.gap-3 {
    flex-direction: column;
    gap: 0.75rem !important;
  }

  .flex-fill {
    width: 100%;
  }
}

@media (max-width: 480px) {
  .modern-tabs {
    width: 100%;
    justify-content: center;
  }

  .tab-button {
    flex: 1;
    justify-content: center;
  }

  /* Ensure mobile content is scrollable */
  .events-content {
    padding: 0.5rem;
  }
}
</style>
