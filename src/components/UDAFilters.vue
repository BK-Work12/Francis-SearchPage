<template>
  <div class="filter-sidebar">
    <div class="sidebar-header">
      <div class="header-top">
        <h3>UDA Configurations</h3>
        <button class="clear-btn" @click="resetFilters">Reset All</button>
      </div>
      
      <div class="search-container">
        <input 
          type="text" 
          v-model="searchQuery" 
          placeholder="Search 32 filters..." 
          class="search-input"
        />
      </div>
    </div>

    <div class="filter-list">
      <div 
        v-for="filter in filteredFilters" 
        :key="filter.key" 
        class="filter-card"
      >
        <div class="filter-info">
          <span class="filter-label">{{ cleanLabel(filter.label) }}</span>
        </div>
        
        <div class="tri-state-toggle">
          <button 
            :class="{ active: filterStates[filter.key] === null }" 
            @click="filterStates[filter.key] = null"
          >Any</button>
          <button 
            :class="{ active: filterStates[filter.key] === false }" 
            @click="filterStates[filter.key] = false"
          >No</button>
          <button 
            :class="{ active: filterStates[filter.key] === true }" 
            @click="filterStates[filter.key] = true"
          >Yes</button>
        </div>
      </div>
      
      <div v-if="filteredFilters.length === 0" class="no-results">
        No filters matching "{{ searchQuery }}"
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'

const searchQuery = ref('')

// 1. Define only the metadata (Key and Label)
const UDA_CONFIG = [
  { key: 'autoCreateClaimRecord', label: 'UDA-Auto Create Claim Record' },
  { key: 'autoUpdateMarkAsReceived', label: 'UDA-Auto-update the Mark As Received' },
  { key: 'copy01to05Inspections', label: 'UDA-Copy 01 to 05 Inspections' },
  { key: 'driverCantRemoveVehicles', label: 'UDA-Driver-Cant-Remove-Vehicles' },
  { key: 'preFillDeliveryBase', label: 'UDA-PreFillDeliveryBase' },
  { key: 'showDriverPayInternalLoads', label: 'UDA-Show Driver Pay for Internal Loads' },
  { key: 'shuttleOneActiveLoad', label: 'UDA Shuttle: Only Allow One Active Load Per Driver' },
  { key: 'truckLoadedRequired', label: 'UDA-Truck Loaded Required' },
  { key: 'allowDriverAddLocationContacts', label: 'UDA-Allow Driver to Add New Location Contacts' },
  { key: 'defaultAttended', label: 'UDA-DefaultAttended' },
  { key: 'autoInspForm', label: 'UDA-Auto Insp form' },
  { key: 'cameraOnlyPhotos', label: 'UDA-Camera-Only-Photos' },
  { key: 'damagePhotoReqNew', label: 'UDA-Require-Damage-Photos-New' },
  { key: 'damagePhotoReqUsed', label: 'UDA-Require-Damage-Photos-Used' },
  { key: 'displayMaxMi', label: 'UDA-DisplayMaxMi' },
  { key: 'hideVINonValidation', label: 'UDA-HideVINonValidation' },
  { key: 'docUploadTypeReqNew', label: 'UDA-Document-Upload-Type-Required-New' },
  { key: 'docUploadTypeReqUsed', label: 'UDA-Document-Upload-Type-Required-Used' },
  { key: 'inspChgAllowed', label: 'UDA-InspChgAllowed' },
  { key: 'forcedInsp', label: 'UDA-ForcedInsp' },
  { key: 'loadSortOrder', label: 'UDA-Loadsortorder' },
  { key: 'optOutUDA', label: 'Opt Out UDA' },
  { key: 'photoRestrictions', label: 'UDA-Photo-Restrictions' },
  { key: 'photoRestrictionsCustomers', label: 'UDA-Photo-Restrictions-Customers' },
  { key: 'reqHeightChk', label: 'UDA-ReqHeightChk' },
  { key: 'reqInspectionPhotosNew', label: 'UDA-Require-Inspection-Photos-New' },
  { key: 'reqInspectionPhotosUsed', label: 'UDA-Require-Inspection-Photos-Used' },
  { key: 'odometerRequiredNew', label: 'UDA-Odometer-Required-New' },
  { key: 'odometerRequiredUsed', label: 'UDA-Odometer-Required-Used' },
  { key: 'odometerNotReqd', label: 'UDA-ODOnotreqd' },
  { key: 'reqUnitWeatCond', label: 'UDA-ReqUnitWeatCond' },
  { key: 'stopETARequired', label: 'UDA-StopETARequired' }
]

// 2. Initialize the state object dynamically
// Result: { autoCreateClaimRecord: null, ... }
const filterStates = reactive(
  UDA_CONFIG.reduce((acc, curr) => {
    acc[curr.key] = null
    return acc
  }, {})
)

// 3. Logic for Search & Formatting
const filteredFilters = computed(() => {
  return UDA_CONFIG.filter(f => 
    f.label.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const cleanLabel = (label) => {
  return label
    .replace(/^UDA-?/, '') // Remove UDA prefix
    .replace(/([A-Z])/g, ' $1') // Add space before capital letters
    .replace(/-/g, ' ') // Replace dashes with spaces
    .trim()
}

const resetFilters = () => {
  Object.keys(filterStates).forEach(key => filterStates[key] = null)
}
</script>

<style scoped>
.filter-sidebar {
  font-family: sans-serif;
  width: 100%;
  max-width: 480px;
  background: #ffffff;
  border-right: 1px solid #e2e8f0;
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.sidebar-header {
  padding: 1.25rem;
  border-bottom: 1px solid #e2e8f0;
  background: #f8fafc;
}

.header-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.header-top h3 {
  margin: 0;
  font-size: 1.1rem;
  color: #1e293b;
}

.clear-btn {
  background: none;
  border: none;
  color: #7c4dff;
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
}

.search-input {
  width: 100%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.9rem;
}

.filter-list {
  flex: 1;
  overflow-y: auto;
  padding: 0.5rem 1.25rem;
}

.filter-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.85rem 0;
  border-bottom: 1px solid #f1f5f9;
}

.filter-label {
  font-size: 0.85rem;
  color: #475569;
  line-height: 1.4;
  padding-right: 10px;
}

.tri-state-toggle {
  display: flex;
  background: #f1f5f9;
  padding: 2px;
  border-radius: 6px;
  min-width: 130px;
}

.tri-state-toggle button {
  flex: 1;
  border: none;
  background: transparent;
  padding: 4px 0;
  font-size: 0.75rem;
  cursor: pointer;
  border-radius: 4px;
  color: #94a3b8;
}

.tri-state-toggle button.active {
  background: #ffffff;
  color: #7c4dff;
  font-weight: 700;
  box-shadow: 0 1px 2px rgba(0,0,0,0.05);
}

.no-results {
  padding: 2rem;
  text-align: center;
  color: #94a3b8;
  font-size: 0.9rem;
}
</style>