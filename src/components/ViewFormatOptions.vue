<template>
  <div class="view-format-card">
    <div class="header-row">
      <div class="main-toggle">
        <button :class="{ active: viewType === 'Summary' }" @click="viewType = 'Summary'">Summary View</button>
        <button :class="{ active: viewType === 'Detail' }" @click="viewType = 'Detail'">Detail View</button>
      </div>
      
      <div class="style-select">
        <span class="tiny-label">Style</span>
        <select v-model="displayStyle" class="minimal-input">
          <option value="compact">Compact</option>
          <option value="normal">Normal</option>
        </select>
      </div>
    </div>

    <div class="settings-bar">
      <div class="sort-logic">
        <span class="text-label">Sort by</span>
        <select v-model="activeSort1" class="minimal-input">
          <option v-for="opt in allColumnOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
        </select>
        <span class="text-label">then</span>
        <select v-model="activeSort2" class="minimal-input">
          <option v-for="opt in allColumnOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
        </select>
      </div>

      <div v-if="viewType === 'Detail'" class="per-page-logic">
        <span class="text-label">Show</span>
        <select v-model="perPage" class="minimal-input">
          <option v-for="n in [10, 30, 50, 100, 200, 500, 1000]" :key="n" :value="n">{{ n }}</option>
          <option value="All">All</option>
        </select>
      </div>
    </div>

    <div class="checkbox-strip">
      <template v-if="viewType === 'Summary'">
        <label class="checkbox-item"><input type="checkbox" v-model="useGrid" /> <span>Use Grid</span></label>
        <label class="checkbox-item"><input type="checkbox" v-model="showChart" /> <span>Show Chart</span></label>
      </template>
      <template v-else>
        <label class="checkbox-item"><input type="checkbox" v-model="showTotals" /> <span>Totals</span></label>
        <label class="checkbox-item"><input type="checkbox" v-model="showIcons" /> <span>Icons</span></label>
        <label class="checkbox-item"><input type="checkbox" v-model="defaultToggle" /> <span>Toggle</span></label>
        <label class="checkbox-item"><input type="checkbox" v-model="putLastAttribOnNextRow" /> <span>Wrap Last Col</span></label>
      </template>
    </div>

    <div class="column-section">
      <div class="section-header">
        <span class="section-label">Active Columns ({{ activeSelectedList.length }})</span>
        <button class="edit-btn" @click="showManager = true">Manage Columns</button>
      </div>
      
      <div class="chip-container">
        <div v-for="val in activeSelectedList" :key="val" class="chip">
          {{ getLabel(val) }}
          <span class="remove-chip" @click="toggleCol(val)">×</span>
        </div>
        <div v-if="activeSelectedList.length === 0" class="empty-state">No columns selected</div>
      </div>
    </div>

    <Transition name="fade">
      <div v-if="showManager" class="modal-overlay" @click.self="showManager = false">
        <div class="modal-window">
          <div class="modal-header">
            <h3>Select View Columns</h3>
            <input v-model="searchQuery" type="text" placeholder="Search columns (e.g. NetSuite, UDA...)" class="search-bar" />
          </div>
          
          <div class="modal-body">
            <div class="grid-layout">
              <label v-for="col in filteredColumns" :key="col.value" class="grid-item" :class="{ selected: isColSelected(col.value) }">
                <input type="checkbox" :checked="isColSelected(col.value)" @change="toggleCol(col.value)" />
                <span>{{ col.label }}</span>
              </label>
            </div>
          </div>

          <div class="modal-footer">
            <button class="primary-btn" @click="showManager = false">Save Configuration</button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

// State
const viewType = ref('Summary');
const displayStyle = ref('compact');
const showManager = ref(false);
const searchQuery = ref('');

// Config Data
const useGrid = ref(false);
const showChart = ref(false);
const showTotals = ref(false);
const showIcons = ref(false);
const defaultToggle = ref(false);
const perPage = ref(30);
const putLastAttribOnNextRow = ref(false);

const summarySort1 = ref('State');
const summarySort2 = ref('CompanyName');
const detailSort1 = ref('State');
const detailSort2 = ref('CompanyName');

// Complete List of All Options provided in prompt
const allColumnOptions = [
  { value: 'AbilitytoPrint_ID', label: 'Ability to Print' },
  { value: 'ActivationDate_ID', label: 'Activation Date' },
  { value: 'AcctID', label: 'Accounting ID' },
  { value: 'Is3plWireless', label: '3PL Wireless' },
  { value: 'age_InsExp', label: 'Age Insurance Expiration' },
  { value: 'UDAAutoCreateClaimRecord_ID', label: 'Auto Create Claim Record' },
  { value: 'UDAAutoupdatetheMarkAsReceived_ID', label: 'Auto-update the Mark As Received' },
  { value: 'DelaySendPreInvoiceonDelivfor24hours_ID', label: 'AutoBilling Delay 24 Hours' },
  { value: 'SendDeliveryBOLPDF_ID', label: 'AutoBilling Send BOL PDF' },
  { value: 'PreInvoiceSendonShipper_ID', label: 'AutoBilling Send On Shipper' },
  { value: 'SendPreInvoiceonDeliv_ID', label: 'AutoBilling Send Preinv' },
  { value: 'OrderEventAutosend', label: 'Autosend Emails' },
  { value: 'BDDomain', label: 'Business Domain' },
  { value: 'BusinessType', label: 'Business Type/s' },
  { value: 'CarrierCargoTypes_ID', label: 'Carrier Cargo Types' },
  { value: 'CompanyName', label: 'Company Name' },
  { value: 'City', label: 'City/ies' },
  { value: 'State', label: 'State/s' },
  { value: 'Country', label: 'Country/ies' },
  { value: 'GLExportName', label: 'GL Export Name' },
  { value: 'NetSuiteAccountID_ID', label: 'NetSuite Account ID' },
  { value: 'NetSuiteVendorId_ID', label: 'NetSuite Carrier ID' },
  { value: 'NetSuiteCustomerId_ID', label: 'NetSuite Customer ID' },
  { value: 'QBCurr', label: 'QuickBooks Currency' },
  { value: 'SCAC_ID', label: 'SCAC' },
  { value: 'TimeZone', label: 'Time Zone' },
  { value: 'TWICRequired_ID', label: 'TWIC Required' },
  { value: 'UDAOdometerRequiredNew_ID', label: 'UDA Require Odometer - New' },
  { value: 'UDAPhotoRestrictions_ID', label: 'UDA Photo Restrictions' },
  // ... Any others from your list map here
];

const selectedSummaryCols = ref(['State', 'CompanyName']);
const selectedDetailCols = ref(['State', 'CompanyName', 'Status']);

// Computed
const activeSelectedList = computed(() => 
  viewType.value === 'Summary' ? selectedSummaryCols.value : selectedDetailCols.value
);

const activeSort1 = computed({
  get: () => viewType.value === 'Summary' ? summarySort1.value : detailSort1.value,
  set: (val) => { if(viewType.value === 'Summary') summarySort1.value = val; else detailSort1.value = val; }
});

const activeSort2 = computed({
  get: () => viewType.value === 'Summary' ? summarySort2.value : detailSort2.value,
  set: (val) => { if(viewType.value === 'Summary') summarySort2.value = val; else detailSort2.value = val; }
});

const filteredColumns = computed(() => {
  if (!searchQuery.value) return allColumnOptions;
  return allColumnOptions.filter(c => c.label.toLowerCase().includes(searchQuery.value.toLowerCase()));
});

// Methods
const getLabel = (val) => allColumnOptions.find(c => c.value === val)?.label || val;
const isColSelected = (val) => activeSelectedList.value.includes(val);

const toggleCol = (val) => {
  const list = activeSelectedList.value;
  const index = list.indexOf(val);
  if (index > -1) list.splice(index, 1);
  else list.push(val);
};
</script>

<style scoped>
.view-format-card {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 1.25rem;
  font-family: 'Inter', system-ui, sans-serif;
  color: #111827;
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  margin-top: 20px;
}

/* Header & Toggles */
.header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.main-toggle {
  background: #f3f4f6;
  padding: 4px;
  border-radius: 8px;
  display: flex;
}

.main-toggle button {
  border: none;
  background: transparent;
  padding: 6px 16px;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  color: #6b7280;
  transition: all 0.2s;
}

.main-toggle button.active {
  background: #fff;
  color: #4f46e5;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

/* Inputs */
.minimal-input {
  border: 1px solid #d1d5db;
  border-radius: 6px;
  padding: 4px 8px;
  font-size: 0.875rem;
  background: #fff;
  outline: none;
}

.tiny-label { font-size: 0.75rem; font-weight: 600; color: #9ca3af; text-transform: uppercase; margin-right: 8px; }
.text-label { font-size: 0.875rem; color: #4b5563; margin: 0 4px; }

/* Settings Bar */
.settings-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #f3f4f6;
}

.checkbox-strip {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 10px;
  margin-bottom: 1.5rem;
  align-items: center;
}
.grid-item input{
  width: unset;
}

.pill-check {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #f3f4f6;
  border: 1.5px solid #e0e7ef;
  padding: 6px 0 6px 18px;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.18s, border 0.18s, box-shadow 0.18s;
  box-shadow: 0 1px 4px 0 rgba(60,72,100,0.06);
  user-select: none;
  position: relative;
  min-width: 0;
}
.pill-check input[type="checkbox"] {
  accent-color: #6366f1;
  width: 1.1em;
  height: 1.1em;
  margin: 0;
  cursor: pointer;
}
.pill-check span {
  pointer-events: none;
}
.pill-check:hover {
  background: #e0e7ff;
  border-color: #6366f1;
  box-shadow: 0 2px 8px 0 rgba(99,102,241,0.10);
}
.pill-check input[type="checkbox"]:checked + span,
.pill-check input[type="checkbox"]:checked ~ span {
  color: #fff;
}
.pill-check input[type="checkbox"]:checked {
  background: #6366f1;
  border-color: #6366f1;
}
.pill-check input[type="checkbox"]:checked ~ span {
  background: #6366f1;
  color: #fff;
  border-radius: 9999px;
  padding: 2px 10px;
  margin-left: -8px;
}

/* Column Area */
.column-section {
  background: #fcfcfd;
  border: 1px dashed #d1d5db;
  border-radius: 8px;
  padding: 1rem;
}

.section-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.75rem;
}

.section-label { font-size: 0.75rem; font-weight: 700; color: #6b7280; text-transform: uppercase; }
.edit-btn { background: none; border: none; color: #4f46e5; font-size: 0.813rem; font-weight: 600; cursor: pointer; }

.chip-container { display: flex; flex-wrap: wrap; gap: 6px; }
.chip {
  background: #eef2ff;
  color: #4338ca;
  padding: 2px 10px;
  border-radius: 4px;
  font-size: 0.75rem;
  display: flex;
  align-items: center;
  gap: 6px;
}
.remove-chip { cursor: pointer; font-size: 14px; color: #9ca3af; }

/* Checkbox Strip Styles */
.checkbox-strip {
  display: flex;
  gap: 12px;
  margin: 18px 0 12px 0;
  flex-wrap: wrap;
}
.checkbox-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.85rem;
  font-weight: 500;
  color: #64748b;
  background: #f3f4f6;
  border-radius: 6px;
  padding: 4px 12px;
  cursor: pointer;
  transition: background 0.15s;
}
.checkbox-item input[type="checkbox"] {
  accent-color: #7c4dff;
  margin-right: 4px;
}
.checkbox-item:hover {
  background: #ede9fe;
}
.checkbox-item input[type="checkbox"]:checked ~ span {
  color: #7c4dff;
}

@media (prefers-color-scheme: dark) {
  .checkbox-strip {
    background: transparent;
  }
  .checkbox-item {
    background: #23263a;
    color: #a3aed6;
  }
  .checkbox-item:hover {
    background: #23204a;
  }
  .checkbox-item input[type="checkbox"] {
    accent-color: #a78bfa;
  }
  .checkbox-item input[type="checkbox"]:checked ~ span {
    color: #a78bfa;
  }
}

/* Modal */
.modal-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.5);
  display: flex; align-items: center; justify-content: center; z-index: 50;
}
.modal-window {
  background: #fff; width: 90%; max-width: 600px; max-height: 80vh;
  border-radius: 12px; display: flex; flex-direction: column; overflow: hidden;
}
.modal-header { padding: 1.25rem; border-bottom: 1px solid #f3f4f6; }
.search-bar { width: 100%; margin-top: 10px; padding: 8px; border: 1px solid #d1d5db; border-radius: 6px; }
.modal-body { flex: 1; overflow-y: auto; padding: 1.25rem; }
.grid-layout { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 10px; }
.grid-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.875rem;
  cursor: pointer;
  padding: 6px 0 6px 18px;
  background: #f3f4f6;
  border: 1.5px solid #e0e7ef;
  border-radius: 9999px;
  min-width: 0;
}
.grid-item.selected { color: #4f46e5; font-weight: 500; }
.modal-footer { padding: 1rem; border-top: 1px solid #f3f4f6; text-align: right; }
.primary-btn { background: #4f46e5; color: #fff; border: none; padding: 8px 20px; border-radius: 6px; cursor: pointer; }
.checkbox-strip input{
  width: unset;
}
@media (prefers-color-scheme: dark) {
  .view-format-card {
    background: #181a20;
    color: #e0e6f5;
  }
  .header-row, .settings-bar {
    background: #23263a;
  }
  .main-toggle button {
    background: #23263a;
    color: #a3aed6;
    border-color: #353a50;
  }
  .main-toggle button.active {
    background: #23204a;
    color: #a78bfa;
    border-color: #a78bfa;
  }
  .style-select, .sort-logic, .per-page-logic {
    color: #a3aed6;
  }
  .tiny-label, .text-label {
    color: #a3aed6;
  }
  .minimal-input {
    background: #23263a;
    color: #e0e6f5;
    border-color: #353a50;
  }
  .minimal-input option {
    background: #23263a;
    color: #e0e6f5;
  }
  .pill-check span {
    color: #e0e6f5;
  }
  .pill-check input[type="checkbox"]:checked ~ span {
    background: #a78bfa;
    color: #23263a;
  }
  .column-section {
    background: #23263a;
    border-color: #353a50;
  }
  .section-header {
    color: #a3aed6;
  }
  .section-label {
    color: #a3aed6;
  }
  .edit-btn {
    color: #a78bfa;
  }
  .chip {
    background: #23204a;
    color: #a78bfa;
  }
  .remove-chip {
    color: #f472b6;
  }
  .modal-overlay {
    background: rgba(0,0,0,0.7);
  }
  .modal-window {
    background: #23263a;
    color: #e0e6f5;
  }
  .modal-header, .modal-footer {
    border-color: #353a50;
  }
  .search-bar {
    background: #23263a;
    color: #e0e6f5;
    border-color: #353a50;
  }
  .grid-item {
    background: #23263a;
    border-color: #353a50;
    color: #e0e6f5;
  }
  .grid-item.selected {
    color: #a78bfa;
    background: #23204a;
  }
  .primary-btn {
    background: #a78bfa;
    color: #23263a;
  }
}

.fade-enter-active, .fade-leave-active { transition: opacity 0.2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>