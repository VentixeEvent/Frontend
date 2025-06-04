<script setup>
import { ref, computed } from "vue";

// Sample booking data based on the image
const bookings = ref([
  {
    id: "INV10011",
    date: "2029/02/15",
    time: "10:30 AM",
    name: "Jackson Moore",
    event: "Symphony Under the Stars",
    category: "Music",
    ticketCategory: "Diamond",
    price: 50,
    qty: 2,
    amount: 100,
    status: "Confirmed",
    voucher: "123456-MUSIC",
  },
  {
    id: "INV10012",
    date: "2029/02/16",
    time: "03:45 PM",
    name: "Alicia Smithson",
    event: "Runway Revolution 2024",
    category: "Fashion",
    ticketCategory: "Platinum",
    price: 120,
    qty: 1,
    amount: 120,
    status: "Pending",
    voucher: "",
  },
  {
    id: "INV10013",
    date: "2029/02/17",
    time: "01:15 PM",
    name: "Natalie Johnson",
    event: "Global Wellness Summit",
    category: "Beauty & Wellness",
    ticketCategory: "CAT 1",
    price: 80,
    qty: 3,
    amount: 240,
    status: "Confirmed",
    voucher: "789101-WELLNESS",
  },
  {
    id: "INV10014",
    date: "2029/02/18",
    time: "09:00 PM",
    name: "Patrick Cooper",
    event: "Champions League Screening Night",
    category: "Sports",
    ticketCategory: "CAT 3",
    price: 30,
    qty: 4,
    amount: 120,
    status: "Cancelled",
    voucher: "",
  },
  {
    id: "INV10015",
    date: "2029/02/18",
    time: "05:30 PM",
    name: "Gloria Ramos",
    event: "Artistry Unveiled: Modern Art Expo",
    category: "Art & Design",
    ticketCategory: "Silver",
    price: 25,
    qty: 2,
    amount: 50,
    status: "Confirmed",
    voucher: "202324-ART",
  },
  {
    id: "INV10016",
    date: "2029/02/19",
    time: "12:00 PM",
    name: "Clara Simmons",
    event: "Tech Future Expo",
    category: "Technology",
    ticketCategory: "CAT 2",
    price: 75,
    qty: 2,
    amount: 150,
    status: "Confirmed",
    voucher: "564738-TECH",
  },
  {
    id: "INV10017",
    date: "2029/02/20",
    time: "02:30 PM",
    name: "Daniel White",
    event: "Culinary Delights Festival",
    category: "Food & Culinary",
    ticketCategory: "Gold",
    price: 60,
    qty: 1,
    amount: 60,
    status: "Cancelled",
    voucher: "928874-CULINARY",
  },
  {
    id: "INV10018",
    date: "2029/02/21",
    time: "06:00 PM",
    name: "Natalie Johnson",
    event: "Echo Beats Festival",
    category: "Music",
    ticketCategory: "Platinum",
    price: 70,
    qty: 3,
    amount: 210,
    status: "Pending",
    voucher: "",
  },
]);

// Filter states
const activeFilter = ref("All");
const searchQuery = ref("");
const categoryFilter = ref("All Category");
const monthFilter = ref("This Month");

// Filtering logic
const filteredBookings = computed(() => {
  let filtered = bookings.value;

  // Status filter
  if (activeFilter.value !== "All") {
    filtered = filtered.filter(
      (booking) => booking.status === activeFilter.value
    );
  }

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    filtered = filtered.filter(
      (booking) =>
        booking.name.toLowerCase().includes(query) ||
        booking.event.toLowerCase().includes(query) ||
        booking.id.toLowerCase().includes(query)
    );
  }

  return filtered;
});

// Status counts
const statusCounts = computed(() => {
  const counts = {
    All: bookings.value.length,
    Confirmed: bookings.value.filter((b) => b.status === "Confirmed").length,
    Pending: bookings.value.filter((b) => b.status === "Pending").length,
    Cancelled: bookings.value.filter((b) => b.status === "Cancelled").length,
  };
  return counts;
});

// Pagination
const currentPage = ref(1);
const itemsPerPage = 8;
const totalItems = computed(() => filteredBookings.value.length);
const totalPages = computed(() => Math.ceil(totalItems.value / itemsPerPage));

const paginatedBookings = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredBookings.value.slice(start, end);
});

// Status badge styling
const getStatusBadgeClass = (status) => {
  switch (status) {
    case "Confirmed":
      return "bg-success text-white";
    case "Pending":
      return "bg-warning text-dark";
    case "Cancelled":
      return "bg-secondary text-white";
    default:
      return "bg-light text-dark";
  }
};

// Ticket category badge styling
const getTicketCategoryClass = (category) => {
  switch (category) {
    case "Diamond":
      return "text-primary";
    case "Platinum":
      return "text-secondary";
    case "Gold":
      return "text-warning";
    case "Silver":
      return "text-muted";
    default:
      return "text-dark";
  }
};

const getTicketCategoryIcon = (category) => {
  switch (category) {
    case "Diamond":
      return "bi-gem";
    case "Platinum":
      return "bi-circle-fill";
    case "Gold":
      return "bi-circle-fill";
    case "Silver":
      return "bi-circle-fill";
    default:
      return "bi-circle";
  }
};
</script>

<template>
  <div class="bookings-container p-4 bg-light rounded-3">
    <!-- Header with filter tabs -->
    <div
      class="d-flex flex-wrap justify-content-between align-items-center mb-4"
    >
      <!-- Status filter tabs -->
      <div class="d-flex mb-3 mb-md-0">
        <button
          v-for="(count, status) in statusCounts"
          :key="status"
          class="btn rounded-pill px-4 py-2 me-2"
          :class="
            activeFilter === status ? 'btn-primary' : 'btn-outline-secondary'
          "
          @click="activeFilter = status"
        >
          <span class="body-14-medium">{{ status }}</span>
          <span class="badge bg-light text-secondary ms-1 rounded-pill">{{
            count
          }}</span>
        </button>
      </div>

      <!-- Search and filters -->
      <div class="d-flex flex-wrap align-items-center">
        <div class="position-relative me-2 mb-2 mb-md-0">
          <input
            type="text"
            class="form-control ps-4"
            placeholder="Search name, event, etc..."
            v-model="searchQuery"
            style="min-width: 250px"
          />
          <i
            class="bi bi-search position-absolute text-muted"
            style="left: 10px; top: 50%; transform: translateY(-50%)"
          ></i>
        </div>

        <div class="dropdown me-2 mb-2 mb-md-0">
          <button
            class="btn btn-outline-secondary dropdown-toggle d-flex align-items-center"
            type="button"
            data-bs-toggle="dropdown"
          >
            {{ categoryFilter }}
          </button>
          <ul class="dropdown-menu">
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="categoryFilter = 'All Category'"
                >All Category</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="categoryFilter = 'Music'"
                >Music</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="categoryFilter = 'Fashion'"
                >Fashion</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="categoryFilter = 'Technology'"
                >Technology</a
              >
            </li>
          </ul>
        </div>

        <div class="dropdown me-2 mb-2 mb-md-0">
          <button
            class="btn btn-outline-secondary dropdown-toggle d-flex align-items-center"
            type="button"
            data-bs-toggle="dropdown"
          >
            {{ monthFilter }}
          </button>
          <ul class="dropdown-menu">
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="monthFilter = 'This Month'"
                >This Month</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="monthFilter = 'Last Month'"
                >Last Month</a
              >
            </li>
            <li>
              <a
                class="dropdown-item"
                href="#"
                @click.prevent="monthFilter = 'All Time'"
                >All Time</a
              >
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Bookings Table -->
    <div class="table-responsive bg-white rounded-3 shadow-sm">
      <table class="table table-hover mb-0">
        <thead class="table-light">
          <tr class="border-bottom">
            <th class="body-12-semibold text-muted px-4 py-3">Invoice ID</th>
            <th class="body-12-semibold text-muted px-4 py-3">Date</th>
            <th class="body-12-semibold text-muted px-4 py-3">Name</th>
            <th class="body-12-semibold text-muted px-4 py-3">Event</th>
            <th class="body-12-semibold text-muted px-4 py-3">
              Ticket Category
            </th>
            <th class="body-12-semibold text-muted px-4 py-3">Price</th>
            <th class="body-12-semibold text-muted px-4 py-3">Qty</th>
            <th class="body-12-semibold text-muted px-4 py-3">Amount</th>
            <th class="body-12-semibold text-muted px-4 py-3">Status</th>
            <th class="body-12-semibold text-muted px-4 py-3">E-Voucher</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="booking in paginatedBookings"
            :key="booking.id"
            class="border-bottom border-light"
          >
            <td class="px-4 py-3">
              <span class="body-14 text-secondary">{{ booking.id }}</span>
            </td>
            <td class="px-4 py-3">
              <div class="body-14">{{ booking.date }}</div>
              <div class="body-12 text-muted">{{ booking.time }}</div>
            </td>
            <td class="px-4 py-3">
              <span class="body-14">{{ booking.name }}</span>
            </td>
            <td class="px-4 py-3">
              <div class="body-14 fw-medium">{{ booking.event }}</div>
              <div class="body-12 text-muted">{{ booking.category }}</div>
            </td>
            <td class="px-4 py-3">
              <div class="d-flex align-items-center">
                <i
                  class="me-2"
                  :class="[
                    getTicketCategoryIcon(booking.ticketCategory),
                    getTicketCategoryClass(booking.ticketCategory),
                  ]"
                ></i>
                <span class="body-14">{{ booking.ticketCategory }}</span>
              </div>
            </td>
            <td class="px-4 py-3">
              <span class="body-14">${{ booking.price }}</span>
            </td>
            <td class="px-4 py-3">
              <span class="body-14">{{ booking.qty }}</span>
            </td>
            <td class="px-4 py-3">
              <span class="body-14 fw-medium">${{ booking.amount }}</span>
            </td>
            <td class="px-4 py-3">
              <span
                class="badge rounded-pill body-12"
                :class="getStatusBadgeClass(booking.status)"
              >
                {{ booking.status }}
              </span>
            </td>
            <td class="px-4 py-3">
              <span class="body-12 text-muted">{{
                booking.voucher || "-"
              }}</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="d-flex justify-content-between align-items-center mt-4">
      <div class="results-count">
        <span class="body-12 text-muted">
          Showing <b>{{ itemsPerPage }}</b> out of <b>{{ totalItems }}</b>
        </span>
      </div>

      <nav aria-label="Page navigation">
        <ul class="pagination mb-0">
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <a
              class="page-link"
              href="#"
              @click.prevent="currentPage = Math.max(1, currentPage - 1)"
            >
              <span>&laquo;</span>
            </a>
          </li>
          <li
            v-for="page in totalPages"
            :key="page"
            class="page-item"
            :class="{ active: currentPage === page }"
          >
            <a class="page-link" href="#" @click.prevent="currentPage = page">{{
              page
            }}</a>
          </li>
          <li
            class="page-item"
            :class="{ disabled: currentPage === totalPages }"
          >
            <a
              class="page-link"
              href="#"
              @click.prevent="
                currentPage = Math.min(totalPages, currentPage + 1)
              "
            >
              <span>&raquo;</span>
            </a>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>

<style scoped>
.bookings-container {
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

.table th {
  border-bottom: 2px solid var(--grey-30);
  background-color: var(--grey-20);
}

.table td {
  border-bottom: 1px solid var(--grey-30);
  vertical-align: middle;
}

.table-hover tbody tr:hover {
  background-color: var(--grey-20);
}

.page-link {
  color: var(--secondary-100);
  border-color: var(--grey-40);
}

.page-item.active .page-link {
  background-color: var(--primary-100);
  border-color: var(--primary-100);
  color: white;
}

.page-link:hover {
  background-color: var(--grey-20);
  border-color: var(--grey-40);
  color: var(--secondary-100);
}

.form-control:focus {
  border-color: var(--secondary-60);
  box-shadow: 0 0 0 0.25rem rgba(55, 67, 125, 0.25);
}

.text-primary {
  color: var(--primary-100) !important;
}

.text-secondary {
  color: var(--secondary-100) !important;
}
</style>
