<script setup>
import { ref, onMounted, watch, computed, onUnmounted } from "vue";
import axios from "axios";
import FiltersModal from "./components/FiltersModal.vue";
import NewOrderForm from "./components/NewOrderForm.vue";

const showModal = ref(false);
const appliedFilters = ref({});
const categoryIdToNameMap = ref({});
const orders = ref([]);
const sortedOrders = computed(() => {
  return [...orders.value].sort((a, b) => {
    return typeof a[sortBy.value] == "number"
      ? a[sortBy.value] - b[sortBy.value]
      : a[sortBy.value].localeCompare(b[sortBy.value]);
  });
});
const sortBy = ref("object_id");

const fetchOrdersIntervalId = ref("");

const setFilters = (filters) => {
  let newFilters = { ...filters };

  if (newFilters.category?.includes("Others")) {
    newFilters.category = [
      ...newFilters.category,
      ...Object.values(categoryIdToNameMap.value).filter(
        (category) => !["Electronics", "Furniture"].includes(category)
      ),
    ];
  }
  appliedFilters.value = newFilters;
};

const openFilterModal = () => {
  showModal.value = true;
};

const closeFilterModal = () => {
  showModal.value = false;
};

const fetchCategories = async () => {
  try {
    const response = await axios.get(
      `${process.env.VUE_APP_SERVER_URL}/categories`
    );
    const newCategoryIdToNameMap = {};
    response.data.forEach((category) => {
      newCategoryIdToNameMap[category.object_id] = category.name;
    });
    categoryIdToNameMap.value = newCategoryIdToNameMap;
  } catch (error) {
    console.error("Error fetching categories:", error);
  }
};

const fetchOrders = async () => {
  try {
    let query = new URLSearchParams(appliedFilters.value).toString();
    const response = await axios.get(
      `${process.env.VUE_APP_SERVER_URL}/orders?${query}`
    );
    orders.value = response.data;
  } catch (error) {
    console.error("Error fetching orders:", error);
  }
};

const setSortBy = (newSortBy) => {
  sortBy.value = newSortBy;
};

watch(appliedFilters, fetchOrders, { deep: true });

onMounted(() => {
  fetchCategories();
  fetchOrders();
  fetchOrdersIntervalId.value = setInterval(fetchOrders, 60 * 1000);
});

onUnmounted(() => {
  clearInterval(fetchOrdersIntervalId.value);
});
</script>

<template>
  <div class="container">
    <button class="open-filter-button" @click="openFilterModal">
      Open Filters
    </button>
    <h2>Orders</h2>
    <table>
      <thead>
        <tr>
          <th @click="setSortBy('object_id')">ID</th>
          <th @click="setSortBy('customer_name')">Customer</th>
          <th @click="setSortBy('status')">Status</th>
          <th @click="setSortBy('category_id')">Category</th>
          <th @click="setSortBy('country')">Country</th>
          <th @click="setSortBy('created_date')">Created Date</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="order in sortedOrders" :key="order.object_id">
          <td>{{ order.object_id }}</td>
          <td>{{ order.customer_name }}</td>
          <td>{{ order.status }}</td>
          <td>{{ categoryIdToNameMap[order.category_id] }}</td>
          <td>{{ order.country }}</td>
          <td>{{ new Date(order.created_date).toLocaleDateString() }}</td>
        </tr>
      </tbody>
    </table>
    <NewOrderForm :categoryIdToNameMap="categoryIdToNameMap" />
    <FiltersModal
      v-if="showModal"
      :setFilters="setFilters"
      @close="closeFilterModal"
    />
  </div>
</template>

<style>
.container {
  max-width: 800px;
  margin: 20px auto;
  font-family: Arial, sans-serif;
  display: flex;
  flex-direction: column;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.open-filter-button {
  margin: 20px 20px 20px auto;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background: #f4f4f4;
}
</style>
