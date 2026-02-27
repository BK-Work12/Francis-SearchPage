<template>
  <div class="view-config-heading">View Format Options</div>
  <div class="view-config-container" :class="{ 'is-dark': isDarkMode }">
    <div class="config-trigger-bar" @click="isExpanded = !isExpanded">
      <div class="trigger-left">
        <span class="view-badge">{{ viewType }} View</span>
        <!-- <span class="col-count">{{ activeSelectedList.length }} Columns</span> -->
      </div>
      <div class="trigger-right">
        <span class="expand-label">{{ isExpanded ? 'Collapse' : 'Configure' }}</span>
        <button class="icon-btn" :class="{ rotated: isExpanded }">▼</button>
      </div>
    </div>

    <Transition name="slide-fade">
      <div v-if="isExpanded" class="config-panel">
        
        <div class="config-row main-controls">
          <div class="control-group">
            <label class="tiny-label">Mode</label>
            <div class="mini-toggle">
              <button :class="{ active: viewType === 'Summary' }" @click="viewType = 'Summary'">Summary</button>
              <button :class="{ active: viewType === 'Detail' }" @click="viewType = 'Detail'">Detail</button>
            </div>
          </div>

          <div class="control-group">
            <label class="tiny-label">Style</label>
            <select v-model="displayStyle" class="compact-select">
              <option value="compact">Compact</option>
              <option value="normal">Normal</option>
            </select>
          </div>

          <div class="control-group">
            <label class="tiny-label">Sort Order</label>
            <div class="sort-pair">
              <select v-model="activeSort1" class="compact-select">
                <option v-for="opt in allColumnOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
              </select>
              <span class="connector">then</span>
              <select v-model="activeSort2" class="compact-select">
                <option v-for="opt in allColumnOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
              </select>
            </div>
          </div>

          <div v-if="viewType === 'Detail'" class="control-group">
            <label class="tiny-label">Per Page</label>
            <select v-model="perPage" class="compact-select width-auto">
              <option v-for="n in [10, 30, 50, 100, 200, 500, 1000]" :key="n" :value="n">{{ n }}</option>
              <option value="All">All</option>
            </select>
          </div>
        </div>

        <div class="config-row wrap">
          <template v-if="viewType === 'Summary'">
            <label class="mini-check"><input type="checkbox" v-model="useGrid" /> <span>Use Grid</span></label>
            <label class="mini-check"><input type="checkbox" v-model="showChart" /> <span>Show Chart</span></label>
          </template>
          <template v-else>
            <label class="mini-check"><input type="checkbox" v-model="showTotals" /> <span>Totals</span></label>
            <label class="mini-check"><input type="checkbox" v-model="showIcons" /> <span>Icons</span></label>
            <label class="mini-check"><input type="checkbox" v-model="defaultToggle" /> <span>Toggle</span></label>
            <label class="mini-check"><input type="checkbox" v-model="putLastAttribOnNextRow" /> <span>Wrap Last Col</span></label>
          </template>
          
          <div class="spacer"></div>
          
          <button class="manage-btn" @click="showManager = true">
            Manage Columns ⚙️
          </button>
        </div>

        <div class="active-columns-tray">
          <div v-for="val in activeSelectedList" :key="val" class="mini-chip">
            {{ getLabel(val) }}
            <span class="remove" @click="toggleCol(val)">×</span>
          </div>
          <div v-if="activeSelectedList.length === 0" class="empty-text">No columns selected. Click "Manage" to add some.</div>
        </div>
      </div>
    </Transition>

    <Transition name="fade">
      <div v-if="showManager" class="modal-overlay" @click.self="showManager = false">
        <div class="modal-window">
          <div class="modal-header">
            <div class="modal-title-row">
              <h3>Select Columns</h3>
              <div class="bulk-actions">
                <button @click="selectAll">All</button>
                <button @click="clearAll">None</button>
              </div>
            </div>
            <input v-model="searchQuery" type="text" placeholder="Search (NetSuite, UDA, etc...)" class="search-bar" />
          </div>
          
          <div class="modal-body">
            <div class="column-grid">
              <label v-for="col in filteredColumns" :key="col.value" class="grid-item" :class="{ selected: isColSelected(col.value) }">
                <input type="checkbox" :checked="isColSelected(col.value)" @change="toggleCol(col.value)" />
                <span>{{ col.label }}</span>
              </label>
            </div>
          </div>

          <div class="modal-footer">
            <button class="primary-btn" @click="showManager = false">Save Changes</button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const isExpanded = ref(false);
const isDarkMode = ref(false); // Can be linked to a global theme provider

// View State
const viewType = ref('Summary');
const displayStyle = ref('compact');
const showManager = ref(false);
const searchQuery = ref('');

// Logic Settings
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

// 100% Complete Column List Restored
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
  { value: 'UDAPhotoRestrictions_ID', label: 'UDA Photo Restrictions' }
];

const selectedSummaryCols = ref(['State', 'CompanyName']);
const selectedDetailCols = ref(['State', 'CompanyName', 'City']);

// Computed Logic
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
  const q = searchQuery.value.toLowerCase();
  return allColumnOptions.filter(c => c.label.toLowerCase().includes(q) || c.value.toLowerCase().includes(q));
});

// Helper Methods
const getLabel = (val) => allColumnOptions.find(c => c.value === val)?.label || val;
const isColSelected = (val) => activeSelectedList.value.includes(val);

const toggleCol = (val) => {
  const list = activeSelectedList.value;
  const index = list.indexOf(val);
  if (index > -1) list.splice(index, 1);
  else list.push(val);
};

const selectAll = () => {
  const allVals = filteredColumns.value.map(c => c.value);
  if (viewType.value === 'Summary') selectedSummaryCols.value = [...new Set([...selectedSummaryCols.value, ...allVals])];
  else selectedDetailCols.value = [...new Set([...selectedDetailCols.value, ...allVals])];
};

const clearAll = () => {
  if (viewType.value === 'Summary') selectedSummaryCols.value = [];
  else selectedDetailCols.value = [];
};
</script>

<style scoped>
/* Container & Base */
.view-config-container {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
  font-family: 'Inter', -apple-system, sans-serif;
  margin: 1rem 0;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
}
.grid-item input{
  width: unset;
}

/* Header Bar */
.config-trigger-bar {
  padding: 10px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  background: #f8fafc;
  transition: background 0.2s;
}
.config-trigger-bar:hover { background: #f1f5f9; }

.view-badge { font-weight: 700; color: #1e293b; font-size: 0.85rem; margin-right: 8px; }
.col-count { font-size: 0.7rem; color: #faa643; background: #fff8ef; padding: 2px 8px; border-radius: 12px; border: 1px solid #ffedd5; }
.expand-label { font-size: 0.75rem; color: #94a3b8; margin-right: 8px; font-weight: 500; }
.icon-btn { border: none; background: none; font-size: 8px; transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1); color: #64748b; }
.icon-btn.rotated { transform: rotate(180deg); }

/* Main Panel */
.config-panel { padding: 16px; border-top: 1px solid #f1f5f9; background: #fff; }
.config-row { display: flex; align-items: flex-end; gap: 20px; margin-bottom: 16px; }
.config-row.wrap { flex-wrap: wrap; margin-top: 12px; padding-top: 12px; border-top: 1px dashed #e2e8f0; align-items: center; }
.spacer { flex: 1; }

.control-group { display: flex; flex-direction: column; gap: 6px; }
.tiny-label { font-size: 10px; font-weight: 800; color: #94a3b8; text-transform: uppercase; letter-spacing: 0.05em; }

/* Inputs */
.compact-select {
  padding: 5px 10px;
  font-size: 0.8rem;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  background: #fff;
  color: #334155;
  outline: none;
}
.compact-select:focus { border-color: #faa643; }
.sort-pair { display: flex; align-items: center; gap: 6px; }
.connector { font-size: 0.7rem; color: #94a3b8; font-weight: 500; }

/* View Toggle */
.mini-toggle { background: #f1f5f9; padding: 3px; border-radius: 6px; display: flex; gap: 2px; }
.mini-toggle button {
  border: none; background: none; padding: 4px 12px; font-size: 0.75rem; border-radius: 4px; cursor: pointer; color: #64748b; font-weight: 600;
}
.mini-toggle button:focus{
  outline: none;
}
.mini-toggle button.active { background: #fff; color: #faa643; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }

/* Checkboxes & Chips */
.mini-check { display: flex; align-items: center; gap: 6px; font-size: 0.8rem; color: #475569; cursor: pointer; font-weight: 500; }
.mini-check input { accent-color: #faa643; width: 14px; height: 14px; }

.active-columns-tray { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 4px; }
.mini-chip { 
  font-size: 11px; background: #f8fafc; color: #475569; padding: 3px 8px; border-radius: 6px; 
  display: flex; align-items: center; gap: 6px; border: 1px solid #e2e8f0; font-weight: 500;
}
.mini-chip:hover { border-color: #faa643; color: #faa643; }
.remove { cursor: pointer; color: #94a3b8; font-size: 14px; line-height: 1; }
.empty-text { font-size: 0.75rem; color: #94a3b8; font-style: italic; }
.manage-btn { background: #fff; border: 1px solid #faa643; color: #faa643; padding: 4px 12px; border-radius: 6px; font-size: 0.75rem; font-weight: 700; cursor: pointer; transition: all 0.2s; }
.manage-btn:hover { background: #faa643; color: #fff; }

/* Modal */
.modal-overlay { position: fixed; inset: 0; background: rgba(15, 23, 42, 0.6); backdrop-filter: blur(2px); display: flex; align-items: center; justify-content: center; z-index: 999; }
.modal-window { background: #fff; width: 95%; max-width: 600px; border-radius: 12px; max-height: 85vh; display: flex; flex-direction: column; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); }
.modal-header { padding: 16px 20px; border-bottom: 1px solid #f1f5f9; }
.modal-title-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.modal-title-row h3 { margin: 0; font-size: 1.1rem; color: #1e293b; }
.bulk-actions { display: flex; gap: 8px; }
.bulk-actions button { background: #f1f5f9; border: none; padding: 2px 8px; border-radius: 4px; font-size: 0.7rem; cursor: pointer; font-weight: 600; color: #64748b; }

.search-bar { width: 100%; max-width: 300px; padding: 10px; border: 1px solid #e2e8f0; border-radius: 8px; outline: none; transition: border 0.2s; }
.search-bar:focus { border-color: #faa643; }

.modal-body { overflow-y: auto; padding: 20px; flex: 1; background: #f8fafc; }
.column-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 10px; }
.grid-item { display: flex; align-items: center; gap: 10px; font-size: 0.85rem; padding: 8px 12px; border-radius: 8px; background: #fff; cursor: pointer; border: 1px solid #e2e8f0; transition: all 0.2s; }
.grid-item.selected { border-color: #faa643; background: #fff8ef; color: #faa643; font-weight: 600; }
.grid-item input { accent-color: #faa643; }

.modal-footer { padding: 16px 20px; border-top: 1px solid #f1f5f9; text-align: right; }
.primary-btn { background: #faa643; color: #fff; border: none; padding: 8px 24px; border-radius: 6px; cursor: pointer; font-weight: 700; box-shadow: 0 2px 4px rgba(250, 166, 67, 0.3); }

/* Transitions */
.slide-fade-enter-active, .slide-fade-leave-active { transition: all 0.3s ease; }
.slide-fade-enter-from, .slide-fade-leave-to { transform: translateY(-10px); opacity: 0; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

/* Dark Mode Overrides */
.is-dark { background: #0f172a; border-color: #1e293b; color: #f1f5f9; }
.is-dark .config-trigger-bar { background: #1e293b; }
.is-dark .view-badge { color: #f1f5f9; }
.is-dark .config-panel { background: #0f172a; border-color: #1e293b; }
.is-dark .compact-select { background: #1e293b; border-color: #334155; color: #f1f5f9; }
.is-dark .mini-toggle { background: #334155; }
.is-dark .mini-chip { background: #1e293b; border-color: #334155; color: #cbd5e1; }
</style>
<style scoped>
.view-config-heading {
  font-size: 1.4rem;
  font-weight: 700;
  color: #147393;
  margin-bottom: 0.5rem;
  margin-top: 0.5rem;
  letter-spacing: 0.01em;
  font-family: 'Inter', -apple-system, sans-serif;
}
</style>