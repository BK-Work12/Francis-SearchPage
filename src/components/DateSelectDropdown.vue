<template>
  <div class="date-select-card">
    <div class="date-select-heading">Select Date Type</div>
    <select 
      class="date-select-select" 
      name="Menu__newDate" 
      id="Menu__newDate"
      v-model="selected"
      @change="onChange"
    >
      <option value="-">No Filter</option>
      <option value="date_filter_ActivationDate_ID" title="Activation Date">Activation Date</option>
      <option value="CallbackDate" title="Callback Date">Callback Date</option>
      <option value="CreatedDate" title="Client Creation Date">Client Creation Date</option>
      <option value="ClientEventDate" title="Contact Status Date">Contact Status Date</option>
      <option value="InsExp" title="Insurance Expiration">Insurance Expiration</option>
      <option value="ProfileChanged" title="Profile Changed">Profile Changed</option>
    </select>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const selected = ref('-');

const emit = defineEmits(['update:dateType']);

function onChange(e) {
  selected.value = e.target.value;
  // Emitting the value so parent components can use the selection
  emit('update:dateType', selected.value);
}
</script>

<style scoped>
.date-select-card {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 2px 16px 0 rgba(60, 72, 88, 0.08);
  border: 1px solid #ececec;
  margin: 2rem 0 1.5rem 0;
  padding: 2rem 1.5rem;
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.date-select-heading {
  font-size: 1.18rem;
  font-weight: 600;
  margin-bottom: 0.7rem;
  color: #7c4dff;
  letter-spacing: 0.01em;
  display: block;
}

.date-select-select {
  width: 100%;
  max-width: 100%;
  min-width: 0;
  padding: 0.6rem 1rem;
  border-radius: 8px;
  border: 1.5px solid #7c4dff;
  font-size: 1rem;
  background: #fafbfc;
  color: #23235b;
  box-sizing: border-box;
  transition: all 0.2s ease-in-out;
  cursor: pointer;
  appearance: none; /* Removes default browser styling for a cleaner look */
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%237c4dff' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 1rem center;
  background-size: 1em;
}

.date-select-select:focus {
  border-color: #23235b;
  background-color: #fff;
  outline: none;
  box-shadow: 0 0 0 3px rgba(124, 77, 255, 0.1);
}

.date-select-select:hover {
  background-color: #f1efff;
}

@media (max-width: 600px) {
  .date-select-card {
    padding: 1.25rem 1rem;
  }
  .date-select-select {
    font-size: 0.95rem;
    padding: 0.5rem 0.7rem;
  }
}
</style>