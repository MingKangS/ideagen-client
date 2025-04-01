<script setup>
import axios from "axios";
import { ref, defineProps } from "vue";

const props = defineProps({
  categoryIdToNameMap: Object,
});

const newOrder = ref({
  customer_name: "",
  category_id: "",
  status: "",
  country: "",
});

const createOrder = async () => {
  console.log(newOrder.value);
  try {
    await axios.post(
      `${process.env.VUE_APP_SERVER_URL}/orders`,
      newOrder.value
    );
    newOrder.value = {
      customer_name: "",
      category_id: "",
      status: "",
      country: "",
    };
  } catch (error) {
    window.alert("An error occurred while creating the order.");
  }
};
</script>

<template>
  <div class="form-container">
    <h2>Create New Order</h2>
    <form @submit.prevent="createOrder">
      <label>Customer Name:</label>
      <input v-model="newOrder.customer_name" type="text" required />

      <label>Category:</label>
      <select v-model="newOrder.category_id" required>
        <option
          v-for="categoryId in Object.keys(props.categoryIdToNameMap)"
          :key="categoryId"
          :value="categoryId"
        >
          {{ props.categoryIdToNameMap[categoryId] }}
        </option>
      </select>

      <label>Status:</label>
      <select v-model="newOrder.status" required>
        <option value="Open">Open</option>
        <option value="Processing">Processing</option>
        <option value="Accepted">Accepted</option>
        <option value="Rejected">Rejected</option>
      </select>

      <label>Country:</label>
      <input v-model="newOrder.country" type="text" required />

      <button type="submit">Create Order</button>
    </form>
  </div>
</template>

<style scoped>
.form-container {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background: #fff;
}
label {
  display: block;
  margin-top: 10px;
}
input,
select,
button {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
}
button {
  margin-top: 15px;
  background: #007bff;
  color: white;
  border: none;
  cursor: pointer;
}
.success {
  color: green;
}
.error {
  color: red;
}
</style>
