<script setup>
import { ref, defineEmits, defineProps, onMounted, watch } from "vue";
import axios from "axios";

const props = defineProps({
  setFilters: Function,
});

defineEmits(["close"]);

const filterValues = ref({
  customerName: [],
  country: [],
});

const filters = ref({
  createdDateStart: "",
  createdDateEnd: "",
  customerName: "",
  status: ["Open", "Processing", "Accepted", "Rejected"],
  category: ["Electronics", "Furniture", "Others"],
  country: "",
});

const isAllCategorySelected = ref(true);
const isAllStatusSelected = ref(true);

const isConfirmCloseModal = ref(false);

const fetchFilterValues = async () => {
  try {
    const response = await axios.get(
      `${process.env.VUE_APP_SERVER_URL}/orders/filters`
    );
    filterValues.value = response.data;
  } catch (error) {
    console.error("Error fetching categories:", error);
  }
};

const applyFilters = () => {
  props.setFilters({ ...filters.value });
};

const onToggleAllStatusSelected = (e) => {
  isAllStatusSelected.value = e.target.checked;
  if (isAllStatusSelected.value) {
    filters.value.status = ["Open", "Processing", "Accepted", "Rejected"];
  } else {
    filters.value.status = [];
  }
};

const onToggleAllCategorySelected = (e) => {
  isAllCategorySelected.value = e.target.checked;
  if (isAllCategorySelected.value) {
    filters.value.category = ["Electronics", "Furniture", "Others"];
  } else {
    filters.value.category = [];
  }
};

const resetFilters = () => {
  filters.value = {
    createdDateStart: "",
    createdDateEnd: "",
    customerName: "",
    status: ["Open", "Processing", "Accepted", "Rejected"],
    category: ["Electronics", "Furniture", "Others"],
    country: "",
  };
};

const onCloseClicked = () => {
  isConfirmCloseModal.value = true;
};

const cancelClose = () => {
  isConfirmCloseModal.value = false;
};

watch(
  filters,
  (newFilters) => {
    localStorage.setItem("filters", JSON.stringify(newFilters));
  },
  { deep: true }
);

onMounted(() => {
  fetchFilterValues();
  const storedFilters = localStorage.getItem("filters");
  if (storedFilters) {
    filters.value = JSON.parse(storedFilters);
  }
});
</script>

<template>
  <div v-if="isConfirmCloseModal" class="modal">
    <div class="modal-content">
      <p>Are you sure you want to navigate away?</p>
      <button @click="$emit('close')">Yes</button>
      <button @click="cancelClose">No</button>
    </div>
  </div>
  <div v-else class="modal">
    <div class="modal-content">
      <h2>Filters</h2>
      <h3>Select criteria filter in listing</h3>

      <div class="filter-group">
        <label>Created Date: </label>Display range from
        <input v-model="filters.createdDateStart" type="text" /> to
        <input v-model="filters.createdDateEnd" type="text" />
      </div>

      <div class="filter-group">
        <label>Customer Name:</label>
        <select v-model="filters.customerName">
          <option
            v-for="name in filterValues.customerName"
            :value="name"
            :key="name"
          >
            {{ name }}
          </option>
        </select>
      </div>

      <div class="filter-group">
        <label>Status:</label>
        <input type="checkbox" @input="onToggleAllStatusSelected" value="All" />
        All <input type="checkbox" v-model="filters.status" value="Open" /> Open
        <input type="checkbox" v-model="filters.status" value="Processing" />
        Processing
        <input type="checkbox" v-model="filters.status" value="Accepted" />
        Accepted
        <input type="checkbox" v-model="filters.status" value="Rejected" />
        Rejected
      </div>

      <div class="filter-group">
        <label>Category:</label>
        <input
          type="checkbox"
          @input="onToggleAllCategorySelected"
          value="All"
        />
        All
        <input type="checkbox" v-model="filters.category" value="Electronics" />
        Electronics
        <input type="checkbox" v-model="filters.category" value="Furniture" />
        Furniture
        <input type="checkbox" v-model="filters.category" value="Others" />
        Others
      </div>

      <div class="filter-group">
        <label>Country:</label>
        <select v-model="filters.country">
          <option
            v-for="country in filterValues.country"
            :value="country"
            :key="country"
          >
            {{ country }}
          </option>
        </select>
      </div>

      <div class="modal-actions">
        <button @click="resetFilters">Reset</button>
        <button @click="applyFilters">Apply</button>
        <button @click="onCloseClicked">Close</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  width: 400px;
  border-radius: 8px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
}

.filter-group {
  margin-bottom: 15px;
}

.modal-actions {
  display: flex;
  justify-content: space-between;
}
</style>
