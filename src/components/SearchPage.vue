
<script setup>
import { ref, watch } from 'vue'
import DateSelectDropdown from './DateSelectDropdown.vue'
import DateRangeDropdown from './DateRangeDropdown.vue'
import DateManualSelect from './DateManualSelect.vue'
import StateSelectDropdown from './StateSelectDropdown.vue'
import OptionTypesRow from './OptionTypesRow.vue'
import ExportOptionsCard from './ExportOptionsCard.vue'
import QuickFiltersBar from './QuickFiltersBar.vue'
import ViewFormatOptions from './ViewFormatOptions.vue'
const search = ref('')
const options = ref({
  restrictTo: false,
  viewFormat: false,
  export: false,
  option: false,
  selections: false,
  dates: false,
})
function openAdvancedFilters() {
  // Logic to open advanced filters drawer
  // You can set a variable here to show the drawer
}
watch(() => options.value.restrictTo, (newVal) => {
  if (newVal) {
    openAdvancedFilters();
  }
});
const focused = ref(false)
function onSearch() {
  // Replace with real search logic
  alert('Searching for: ' + search.value)
}



// View Format Section State
const displayStyle = ref('normal')
const viewType = ref('Summary')
const summarySort1 = ref('')
const summarySort2 = ref('')
const summarySort3 = ref('')
const useGrid = ref(false)
const showChart = ref(false)
const availableSummaryCols = ref([])
const selectedSummaryCols = ref([])
const summaryOptions = [
  { value: 'State', label: 'State/s' },
  { value: 'AcctID', label: 'Accounting ID' },
  { value: 'ActivationDate', label: 'Activation Date' },
  { value: 'CompanyName', label: 'Company Name' },
  // ...add more as needed...
]
const summaryColumns = [
  { value: 'AbilitytoPrint_ID', label: 'Ability to Print' },
  { value: 'ActivationDate_ID', label: 'Activation Date' },
  { value: 'AcctID', label: 'Accounting ID' },
  { value: 'CompanyName', label: 'Company Name' },
  // ...add more as needed...
]
const detailSort1 = ref('')
const detailSort2 = ref('')
const detailSort3 = ref('')
const perPage = ref('30')
const showTotals = ref(false)
const showIcons = ref(false)
const defaultToggle = ref(false)
const putLastAttribOnNextRow = ref(false)
const putPanelAttribOnNextRow = ref(false)
const availableDetailCols = ref([])
const selectedDetailCols = ref([])
const detailOptions = [
  { value: 'Client_Id', label: 'Client ID' },
  { value: 'CompanyName', label: 'Company Name' },
  { value: 'State', label: 'State' },
  // ...add more as needed...
]
const detailColumns = [
  { value: 'Client', label: 'Client' },
  { value: 'ContactMgr', label: 'Contact Mgr' },
  { value: 'FirstName', label: 'Contact First Name' },
  { value: 'LastName', label: 'Contact Last Name' },
  { value: 'Phone', label: 'Phone' },
  { value: 'City', label: 'City' },
  { value: 'State', label: 'State' },
  { value: 'Users', label: 'Users' },
  // ...add more as needed...
]
function getSummaryColLabel(val) {
  const found = summaryColumns.find(opt => opt.value === val)
  return found ? found.label : val
}
function getDetailColLabel(val) {
  const found = detailColumns.find(opt => opt.value === val)
  return found ? found.label : val
}
function moveUp(type) {}
function moveDown(type) {}
function addCol(type) {}
function removeCol(type) {}
</script>


<template>
  <div class="modern-bg">
    <div class="glass search-container">
      <div class="header-row">
        <svg class="logo-icon" viewBox="0 0 48 48" fill="none"><circle cx="24" cy="24" r="22" fill="#7c4dff" fill-opacity="0.15"/><path d="M24 14a10 10 0 1 1 0 20 10 10 0 0 1 0-20zm0 2a8 8 0 1 0 0 16 8 8 0 0 0 0-16z" fill="#7c4dff"/></svg>
        <h1 class="title">Client Search</h1>
      </div>
      <div class="search-bar-row">
        <div class="search-bar-outer" :class="{ focused }">
          <svg class="search-icon" viewBox="0 0 24 24"><path d="M15.5 15.5L20 20" stroke="#7c4dff" stroke-width="2" stroke-linecap="round"/><circle cx="11" cy="11" r="7" stroke="#7c4dff" stroke-width="2"/></svg>
          <input
            v-model="search"
            class="search-input"
            type="text"
            placeholder="Search clients..."
            @focus="focused = true"
            @blur="focused = false"
            @keyup.enter="onSearch"
          />
          <button class="search-btn" @click="onSearch">
            <svg viewBox="0 0 24 24" width="22" height="22"><path d="M5 12h14M12 5l7 7-7 7" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" fill="none"/></svg>
          </button>
        </div>
      </div>
      <div class="options-row">
        <div class="option-group">
          <label v-for="(val, key) in options" :key="key" class="option-label">
            <input type="checkbox" v-model="options[key]" />
            <span>{{ key.charAt(0).toUpperCase() + key.slice(1).replace(/([A-Z])/g, ' $1') }}</span>
          </label>
        </div>
        <a href="#" class="save-tabs">
          <svg viewBox="0 0 20 20" width="18" height="18"><path d="M5 4v12h10V4H5zm2 2h6v8H7V6z" fill="#7c4dff"/></svg>
          <span>Save Tabs</span>
        </a>
      </div>
      <QuickFiltersBar :restrictTo="options.restrictTo" @openAdvancedFilters="openAdvancedFilters" />
      <DateRangeDropdown v-if="options.dates" />
      <StateSelectDropdown v-if="options.selections" />
      <OptionTypesRow v-if="options.option" />
      <ExportOptionsCard v-if="options.export" />
      <!-- View Format Section: open when checked -->
      <ViewFormatOptions v-if="options.viewFormat" />

      <div class="result-btn-row">
        <button class="result-btn show-btn" @click="showResult">Show Result</button>
        <button class="result-btn reset-btn" @click="resetResult">Reset</button>
      </div>
      <div v-if="resultVisible" class="result-section" style="margin-top: 1.5rem;">
        <div v-if="filteredResults.length === 0" class="no-result">No results found.</div>
        <ul v-else class="result-list">
          <li v-for="item in filteredResults" :key="item" class="result-item">{{ item }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>


<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@700;500;400&display=swap');


/* Result Button Row Styles */
          .result-btn-row {
            margin-top: 2rem;
            display: flex;
            gap: 1.2rem;
            justify-content: flex-start;
            align-items: center;
          }
          .result-btn {
            padding: 0.55rem 1.6rem;
            font-size: 1.08rem;
            font-weight: 600;
            border-radius: 8px;
            border: 1.5px solid #ececec;
            background: #f7f8fa;
            color: #646cff;
            cursor: pointer;
            transition: background 0.18s, color 0.18s, border 0.18s;
            box-shadow: 0 1px 6px #e0e0e0;
          }
          .result-btn.show-btn {
            background: #646cff;
            color: #fff;
            border-color: #646cff;
          }
          .result-btn.show-btn:hover {
            background: #5157d8;
            color: #fff;
            border-color: #5157d8;
          }
          .result-btn.reset-btn {
            background: #fff;
            color: #ef4444;
            border-color: #ef4444;
          }
          .result-btn.reset-btn:hover {
            background: #fef2f2;
            color: #b91c1c;
            border-color: #b91c1c;
          }


.modern-bg {
  background: #f5f6fa;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: flex-start;
  font-family: 'Montserrat', Arial, sans-serif;
  overflow-x: hidden;
  width: 100%;
  min-height: 100vh;
  padding: 0;
  box-sizing: border-box;
}



html, body {
  height: 100vh;
  margin: 0;
  padding: 0;
  width: 100%;
  overflow-x: hidden;
  box-sizing: border-box;
}

.glass {
  background: #fff;
  box-shadow: 0 2px 16px 0 rgba(60, 72, 88, 0.08);
  border: 1px solid #ececec;
  padding: 1.5rem 2rem 1rem 2rem;
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
  animation: fadeIn 0.8s cubic-bezier(.4,0,.2,1);
}

.glass.search-container {
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
}
@media (max-width: 900px) {
  .glass {
    max-width: 98vw;
    padding: 1rem 0.5rem 0.5rem 0.5rem;
  }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(40px); }
  to { opacity: 1; transform: none; }
}

.header-row {
  display: flex;
  align-items: center;
  gap: 1.1rem;
  margin-bottom: 2rem;
}
.logo-icon {
  width: 44px;
  height: 44px;
  background: none;
  border-radius: 50%;
  box-shadow: 0 2px 8px #e0e0e0;
}
.title {
  color: #23235b;
  font-size: 1.7rem;
  font-weight: 700;
  letter-spacing: 0.2px;
  margin: 0;
}

.search-bar-row {
  margin-bottom: 1.2rem;
}
.search-bar-outer {
  display: flex;
  align-items: center;
  background: #f7f8fa;
  border-radius: 10px;
  box-shadow: 0 1px 6px #e0e0e0;
  border: 1.2px solid #e0e0e0;
  padding: 0.15rem 0.6rem 0.15rem 0.6rem;
  transition: box-shadow 0.2s, border 0.2s;
  position: relative;
}
.search-bar-outer.focused {
  box-shadow: 0 2px 12px #646cff22;
  border-color: #646cff;
}
.search-icon {
  width: 24px;
  height: 24px;
  margin-right: 0.5rem;
  opacity: 0.6;
}
.search-input {
  flex: 1;
  background: transparent;
  border: none;
  outline: none;
  font-size: 1.08rem;
  color: #23235b;
  padding: 0.6rem 0.2rem;
  font-family: inherit;
  font-weight: 500;
  letter-spacing: 0.1px;
  transition: color 0.2s;
}
.search-input::placeholder {
  color: #b0b3c6;
  font-weight: 400;
}
.search-btn {
  background: #646cff;
  color: #fff;
  border: none;
  border-radius: 7px;
  padding: 0.45rem 1rem;
  margin-left: 0.7rem;
  font-size: 1.05rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: background 0.2s, box-shadow 0.2s;
  box-shadow: 0 1px 6px #e0e0e0;
}
.search-btn:hover {
  background: #5157d8;
  box-shadow: 0 2px 10px #646cff22;
}

.options-row {
  display: flex;
  align-items: center;
  gap: 1.2rem;
  flex-wrap: wrap;
  margin-top: 0.5rem;
  justify-content: flex-start;
  width: 100%;
  max-width: 100%;
  min-width: 0;
  box-sizing: border-box;
  overflow-x: hidden;
}
.option-group {
  display: flex;
  gap: 0.8rem;
  flex-wrap: wrap;
  width: 100%;
  max-width: 100%;
  min-width: 0;
  box-sizing: border-box;
}
.option-group input{
  width: unset;
  margin: 0;
}
.option-label {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  font-size: 0.98rem;
  color: #23235b;
  cursor: pointer;
  user-select: none;
  background: #f7f8fa;
  border-radius: 5px;
  padding: 0.18rem 0.6rem 0.18rem 0.35rem;
  transition: background 0.15s;
  min-width: 0;
  max-width: 100%;
  box-sizing: border-box;
}
@media (max-width: 700px) {
  .options-row {
    flex-direction: column;
    align-items: stretch;
    gap: 0.7rem;
  }
  .option-group {
    flex-direction: column;
    gap: 0.5rem;
  }
  .option-label {
    width: 100%;
    max-width: 100%;
    min-width: 0;
  }
}
.option-label input[type="checkbox"] {
  accent-color: #646cff;
}
.option-label .bold {
  font-weight: 700;
}
.save-tabs {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  color: #646cff;
  font-size: 0.99rem;
  text-decoration: none;
  font-weight: 600;
  background: #f7f8fa;
  border-radius: 6px;
  padding: 0.25rem 0.8rem;
  transition: background 0.15s, color 0.15s;
  margin-left: 1.2rem;
}
.save-tabs:active {
  background: #646cff;
  color: #fff;
}

.date-inputs-card {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 2px 16px 0 rgba(60, 72, 88, 0.08);
  border: 1px solid #ececec;
  margin: 2rem 0 1.5rem 0;
  padding: 2rem 1.5rem;
  width: 100%;
  max-width: 100vw;
  box-sizing: border-box;
}
.date-inputs-row {
  display: flex;
  flex-direction: row;
  gap: 2rem;
  justify-content: flex-start;
  align-items: flex-end;
}
.date-label {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  flex: 1;
}
.date-heading {
  font-size: 1.18rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: #7c4dff;
  letter-spacing: 0.01em;
}

.view-format-section {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 2px 16px 0 rgba(60, 72, 88, 0.08);
  border: 1px solid #ececec;
  margin: 2rem 0 1.5rem 0;
  padding: 2rem 1.5rem;
  width: 100%;
  max-width: 100vw;
  box-sizing: border-box;
  color: #23235b;
  overflow-x: auto;
}

.view-format-heading {
  font-size: 1.25rem;
  font-weight: 700;
  color: #7c4dff;
  margin-bottom: 1.5rem;
  letter-spacing: 0.01em;
}
.display-style-row, .view-type-row, .sort-row, .per-page-row, .checkbox-row {
  display: flex;
  align-items: center;
  gap: 1.2rem;
  margin-bottom: 1.2rem;
}
.columns-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1.2rem;
  margin-top: 1.2rem;
  min-width: 0;
  width: 100%;
}

.columns-select {
  width: 100%;
  min-width: 0;
  min-height: 120px;
  border-radius: 6px;
  border: 1px solid #ececec;
  padding: 6px;
  background: #f7f8fa;
  color: #23235b;
  font-family: inherit;
  font-size: 1rem;
}
.column-buttons {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  justify-content: center;
}
.column-buttons button {
  padding: 4px 10px;
  border-radius: 6px;
  border: 1px solid #ececec;
  background: #f7f8fa;
  color: #646cff;
  font-weight: 600;
  font-family: inherit;
  font-size: 0.98rem;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
}
.column-buttons button:hover {
  background: #646cff;
  color: #fff;
}
</style>
