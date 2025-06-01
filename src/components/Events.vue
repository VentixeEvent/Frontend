<script setup>
import { ref } from "vue";

// Event data for the events list
const events = ref([
  {
    id: 1,
    category: "Outdoor & Adventure",
    title: "Adventure Gear Show",
    description:
      "Top outdoor brands showcase the latest gear. Discounts, demos, and expert consultations.",
    location: "Rocky Ridge Hall, Denver, CO",
    date: "June 5, 2023",
    time: "3:00 PM",
    percentSold: 65,
    ticketsLeft: 115,
    price: "$40",
    image: "adventure.jpg",
  },
  {
    id: 2,
    category: "Food & Culinary",
    title: "Culinary Delights Festival",
    description:
      "Embark on a culinary adventure! Sample delicious dishes, watch chef demos, and savor diverse cuisines.",
    location: "The Plaza, San Francisco, CA",
    date: "May 25, 2023",
    time: "11:00 AM",
    percentSold: 60,
    ticketsLeft: 120,
    price: "$45",
    image: "culinary.jpg",
  },
  {
    id: 3,
    category: "Music",
    title: "Echo Beats Festival",
    description:
      "Dance the night away to top DJs and live electronic music. An unforgettable festival experience",
    location: "Sunset Park, Los Angeles, CA",
    date: "May 20, 2023",
    time: "6:00 PM",
    percentSold: 70,
    ticketsLeft: 55,
    price: "$60",
    image: "music.jpg",
  },
  {
    id: 4,
    category: "Fashion",
    title: "Runway Revolution 2029",
    description:
      "Celebrate emerging talent at Runway Revolution 2029. Discover the next generation of fashion icons.",
    location: "Vogue Hall, New York, NY",
    date: "May 1, 2023",
    time: "8:00 PM",
    percentSold: 100,
    ticketsLeft: 0,
    price: "$30",
    image: "fashion.jpg",
  },
  {
    id: 5,
    category: "Art & Design",
    title: "Artistry Unveiled Expo",
    description:
      "Explore diverse art and design forms. Connect with global artists and discover creative inspiration.",
    location: "Modern Art Gallery, Chicago, IL",
    date: "May 15, 2023",
    time: "10:00 AM",
    percentSold: 85,
    ticketsLeft: 20,
    price: "$20",
    image: "art.jpg",
  },
  {
    id: 6,
    category: "Technology",
    title: "Tech Future Expo",
    description:
      "Explore the latest tech innovations here. Discover trends and solutions shaping tomorrow",
    location: "Silicon Valley, San Jose, CA",
    date: "Jun 1, 2023",
    time: "10:00 AM",
    percentSold: 55,
    ticketsLeft: 85,
    price: "$80",
    image: "tech.jpg",
  },
]);

const currentPage = ref(1);
const eventsPerPage = ref(8);
const totalEvents = 48;

const tabs = ref([
  { name: "Active", count: 44, active: true },
  { name: "Draft", count: 22, active: false },
  { name: "Past", count: 37, active: false },
]);

const searchQuery = ref("");
const selectedCategory = ref("All Category");
const selectedMonth = ref("This Month");
</script>

<template>
  <div class="events-container p-4 bg-light rounded-3">
    <!-- Header with tabs, search, and filters -->
    <div
      class="d-flex flex-wrap justify-content-between align-items-center mb-4"
    >
      <!-- Tabs -->
      <div class="d-flex mb-3 mb-md-0">
        <button
          v-for="tab in tabs"
          :key="tab.name"
          class="btn rounded-pill px-4 py-2 me-2"
          :class="tab.active ? 'btn-primary' : 'btn-outline-secondary'"
        >
          <span class="body-14-medium">{{ tab.name }}</span>
          <span class="badge bg-light text-secondary ms-1 rounded-pill">{{
            tab.count
          }}</span>
        </button>
      </div>

      <!-- Search and filters -->
      <div class="d-flex flex-wrap align-items-center">
        <div class="position-relative me-2 mb-2 mb-md-0">
          <input
            type="text"
            class="form-control ps-4"
            placeholder="Search event, location, etc"
            v-model="searchQuery"
          />
          <i
            class="bi bi-search position-absolute"
            style="left: 10px; top: 50%; transform: translateY(-50%)"
          ></i>
        </div>

        <div class="dropdown me-2 mb-2 mb-md-0">
          <button
            class="btn btn-outline-secondary dropdown-toggle d-flex align-items-center"
            type="button"
            data-bs-toggle="dropdown"
          >
            <i class="bi bi-funnel me-2"></i>
            {{ selectedCategory }}
          </button>
          <ul class="dropdown-menu">
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="selectedCategory = 'All Category'"
                >All Category</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="selectedCategory = 'Music'"
                >Music</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="selectedCategory = 'Food & Culinary'"
                >Food & Culinary</a
              >
            </li>
            <!-- Add more categories as needed -->
          </ul>
        </div>

        <div class="dropdown me-2 mb-2 mb-md-0">
          <button
            class="btn btn-outline-secondary dropdown-toggle d-flex align-items-center"
            type="button"
            data-bs-toggle="dropdown"
          >
            <i class="bi bi-calendar me-2"></i>
            {{ selectedMonth }}
          </button>
          <ul class="dropdown-menu">
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="selectedMonth = 'This Month'"
                >This Month</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="selectedMonth = 'Next Month'"
                >Next Month</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="selectedMonth = 'All Time'"
                >All Time</a
              >
            </li>
          </ul>
        </div>

        <button class="btn btn-outline-secondary">
          <i class="bi bi-grid-3x3-gap"></i>
        </button>

        <button class="btn btn-secondary ms-2">
          <i class="bi bi-list"></i>
        </button>
      </div>
    </div>

    <!-- Events List -->
    <div class="events-list">
      <div
        v-for="event in events"
        :key="event.id"
        class="event-card bg-white rounded-3 mb-3 overflow-hidden shadow-sm"
      >
        <div class="row g-0">
          <div class="col-md-2">
            <div class="bg-light h-100" style="min-height: 120px">
              <!-- Replace with actual image when available -->
              <div class="placeholder-image h-100"></div>
            </div>
          </div>
          <div class="col-md-4 p-3">
            <div class="category-badge mb-1">
              <span
                class="badge rounded-pill body-12"
                :style="{
                  backgroundColor: 'var(--primary-30)',
                  color: 'var(--primary-100)',
                }"
                >{{ event.category }}</span
              >
            </div>
            <h5 class="h5-semibold text-secondary-100 mb-1">
              {{ event.title }}
            </h5>
            <p class="body-12 text-muted mb-2">{{ event.description }}</p>
          </div>
          <div class="col-md-3 p-3 d-flex flex-column justify-content-center">
            <div class="d-flex align-items-center mb-1">
              <i class="bi bi-geo-alt text-muted me-2"></i>
              <span class="body-12 text-muted">{{ event.location }}</span>
            </div>
            <div class="d-flex align-items-center">
              <i class="bi bi-calendar text-muted me-2"></i>
              <span class="body-12 text-muted"
                >{{ event.date }} - {{ event.time }}</span
              >
            </div>
          </div>
          <div class="col-md-3 p-3">
            <div class="row h-100 align-items-center">
              <div class="col-6">
                <div class="ticket-progress">
                  <div class="progress" style="height: 8px">
                    <div
                      class="progress-bar"
                      :style="{
                        width: `${event.percentSold}%`,
                        backgroundColor: 'var(--primary-100)',
                      }"
                      role="progressbar"
                      :aria-valuenow="event.percentSold"
                      aria-valuemin="0"
                      aria-valuemax="100"
                    ></div>
                  </div>
                  <div class="d-flex justify-content-between mt-1">
                    <span class="body-12 text-muted"
                      >{{ event.percentSold }}% <small>Ticket Sold</small></span
                    >
                    <span class="ticket-icon">
                      <i class="bi bi-ticket-perforated"></i>
                    </span>
                  </div>
                </div>

                <div class="mt-2">
                  <div class="tickets-left text-center">
                    <span class="body-16-semibold">{{
                      event.ticketsLeft
                    }}</span>
                    <div class="body-12 text-muted">Tickets Left</div>
                  </div>
                </div>
              </div>
              <div class="col-6 text-end">
                <div class="price-tag">
                  <span class="h4" style="color: var(--primary-100)">{{
                    event.price
                  }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div class="d-flex justify-content-between align-items-center mt-4">
      <div class="results-count">
        <span class="body-12 text-muted"
          >Showing <b>8</b> out of <b>{{ totalEvents }}</b></span
        >
      </div>

      <nav aria-label="Page navigation">
        <ul class="pagination mb-0">
          <li class="page-item">
            <a class="page-link" href="#" aria-label="Previous">
              <span aria-hidden="true">&laquo;</span>
            </a>
          </li>
          <li class="page-item active"><a class="page-link" href="#">1</a></li>
          <li class="page-item"><a class="page-link" href="#">2</a></li>
          <li class="page-item"><a class="page-link" href="#">3</a></li>
          <li class="page-item disabled"><span class="page-link">...</span></li>
          <li class="page-item"><a class="page-link" href="#">8</a></li>
          <li class="page-item">
            <a class="page-link" href="#" aria-label="Next">
              <span aria-hidden="true">&raquo;</span>
            </a>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>

<style scoped>
.events-container {
  background-color: var(--grey-20);
}

.btn-primary {
  background-color: var(--primary-100);
  border-color: var(--primary-100);
}

.btn-outline-secondary {
  color: var(--grey-60);
  border-color: var(--grey-40);
}

.btn-secondary {
  background-color: var(--secondary-100);
  border-color: var(--secondary-100);
}

.placeholder-image {
  background-color: var(--grey-30);
}

.page-link {
  color: var(--secondary-100);
}

.page-item.active .page-link {
  background-color: var(--primary-100);
  border-color: var(--primary-100);
}

.text-secondary-100 {
  color: var(--secondary-100);
}

.ticket-icon {
  color: var(--primary-100);
}

/* For responsive layout */
@media (max-width: 767px) {
  .event-card .row {
    flex-direction: column;
  }
}
</style>
