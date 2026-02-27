<script setup>
import { ref, watch, computed } from 'vue'
import DateRangeDropdown from './DateRangeDropdown.vue'
import StateSelectDropdown from './StateSelectDropdown.vue'
import OptionTypesRow from './OptionTypesRow.vue'
import ExportOptionsCard from './ExportOptionsCard.vue'
import QuickFiltersBar from './QuickFiltersBar.vue'
import ViewFormatOptions from './ViewFormatOptions.vue'

// --- STATE MANAGEMENT ---
const search = ref('')
const focused = ref(false)
const resultVisible = ref(false)
const isSidebarOpen = ref(true)

const options = ref({
  restrictTo: false,
  viewFormat: false,
  export: false,
  option: false,
  selections: false,
  dates: false,
})

// --- MOCK DATA & SEARCH LOGIC ---
const allResults = ['Alpha Dynamics', 'Global Connect', 'Horizon Tech', 'Nexus Solutions', 'Pinnacle Group', 'Quantum Labs']
const filteredResults = computed(() => {
  if (!search.value) return allResults
  return allResults.filter(item => item.toLowerCase().includes(search.value.toLowerCase()))
})

function onSearch() {
  resultVisible.value = true
}

function resetResult() {
  search.value = ''
  resultVisible.value = false
  Object.keys(options.value).forEach(key => options.value[key] = false)
}

function openAdvancedFilters() {
  // Logic for advanced drawer if needed
}

function formatLabel(key) {
  return key.charAt(0).toUpperCase() + key.slice(1).replace(/([A-Z])/g, ' $1')
}

watch(() => options.value.restrictTo, (newVal) => {
  if (newVal) openAdvancedFilters()
})
</script>

<template>
  <div class="dashboard-wrapper">
    <header class="top-nav glass">
      <div class="nav-left">
        <svg class="logo-icon" viewBox="0 0 48 48" fill="none">
          <circle cx="24" cy="24" r="22" fill="#faa643" fill-opacity="0.15"/>
          <path d="M24 14a10 10 0 1 1 0 20 10 10 0 0 1 0-20zm0 2a8 8 0 1 0 0 16 8 8 0 0 0 0-16z" fill="#faa643"/>
        </svg>
        <h1 class="nav-title">Client Search</h1>
      </div>



      <div class="nav-right">
        <button class="icon-btn save-tabs" title="Save Tabs">
          <svg viewBox="0 0 20 20" width="18" height="18"><path d="M5 4v12h10V4H5zm2 2h6v8H7V6z" fill="currentColor"/></svg>
          <span>Save View</span>
        </button>
        <button class="sidebar-toggle" @click="isSidebarOpen = !isSidebarOpen">
          <svg viewBox="0 0 24 24" width="20" height="20" stroke="currentColor" fill="none" stroke-width="2"><path d="M3 12h18M3 6h18M3 18h18"/></svg>
        </button>
      </div>
    </header>

    <div class="main-layout">
      <main class="results-area">
        <div v-if="!resultVisible && !Object.values(options).some(v => v)" class="placeholder-state">
          <div class="pulse-icon">🔍</div>
          <h2>Find your clients</h2>
          <p>Adjust filters on the right and click <strong>Update Results</strong>.</p>
        </div>

            <QuickFiltersBar :restrictTo="options.restrictTo" @openAdvancedFilters="openAdvancedFilters" />
            <DateRangeDropdown v-if="options.dates" />
            <StateSelectDropdown v-if="options.selections" />
            <OptionTypesRow v-if="options.option" />
            <ExportOptionsCard v-if="options.export" />
            <ViewFormatOptions v-if="options.viewFormat" />
  
      </main>

      <aside class="sidebar-right glass" :class="{ 'is-collapsed': !isSidebarOpen }">
        <div class="sidebar-header">
          <!-- <span class="header-label">Filters & Settings</span> -->
          <button class="reset-btn" @click="resetResult">Reset</button>
        </div>

        <div class="sidebar-scroll">
          <div class="filter-box">
            <h4 class="filter-title">Toggle Modules</h4>
            <div class="checkbox-grid">
              <label v-for="(val, key) in options" :key="key" class="custom-check">
                <input type="checkbox" v-model="options[key]" />
                <span class="box"></span>
                <span class="text">{{ formatLabel(key) }}</span>
              </label>
            </div>
          </div>

          <div class="dynamic-group">
       
          </div>
        </div>

        <div class="sidebar-footer">
          <button class="update-btn" @click="onSearch">Update Results</button>
        </div>
      </aside>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700&display=swap');

.dashboard-wrapper {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f9fafb;
  font-family: 'Montserrat', sans-serif;
  color: #1f2937;
  overflow: hidden;
}

/* Glassmorphism utility */
.glass {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border-left: 1px solid #e5e7eb;
}

/* --- TOP NAVIGATION --- */
.top-nav {
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  border-bottom: 1px solid #e5e7eb;
  z-index: 10;
}
.nav-left {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: nowrap;
  white-space: nowrap;
}
.nav-title { font-size: 1.1rem; font-weight: 700; color: #23235b; margin: 0; }

.nav-search-box {
  display: flex;
  align-items: center;
  background: #f3f4f6;
  border-radius: 50px;
  padding: 6px 16px;
  width: 320px;
  transition: all 0.3s ease;
  border: 1.5px solid transparent;
}
.nav-search-box.focused {
  background: #fff;
  width: 420px;
  border-color: #faa643;
  box-shadow: 0 0 0 4px rgba(250, 166, 67, 0.1);
}
.nav-search-box input {
  background: none;
  border: none;
  outline: none;
  padding-left: 10px;
  width: 100%;
  font-family: inherit;
  font-size: 0.9rem;
}

.nav-right { display: flex; align-items: center; gap: 12px; }
.save-tabs { 
  display: flex; align-items: center; gap: 6px;
  background: #fff8ef; color: #faa643;
  border: none; padding: 8px 14px; border-radius: 8px;
  font-weight: 600; font-size: 0.85rem; cursor: pointer;
}
.sidebar-toggle {
  background: none; border: 1px solid #e5e7eb;
  padding: 6px; border-radius: 8px; cursor: pointer;
}

/* --- LAYOUT --- */
.main-layout { display: flex; flex: 1; overflow: hidden; }

/* --- SIDEBAR (RIGHT) --- */
.sidebar-right {
  max-width: 240px;
  width: 100%;
  display: flex;
  flex-direction: column;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.sidebar-right.is-collapsed {
  width: 0;
  overflow: hidden;
  border-left: none;
}

.sidebar-header {
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #f3f4f6;
}
.header-label { font-size: 0.75rem; font-weight: 700; color: #9ca3af; text-transform: uppercase; }
.reset-btn { background: none;padding: 0; border: none; color: #ef4444; font-weight: 600; cursor: pointer; }

.sidebar-scroll { flex: 1; overflow-y: auto; padding: 20px; }
.filter-title { font-size: 0.85rem; font-weight: 700; margin-bottom: 12px; color: #374151; }

.checkbox-grid { display: grid; gap: 10px; }
.custom-check { display: flex; align-items: center; cursor: pointer; font-size: 0.85rem; font-weight: 500; }
.custom-check input { display: none; }
.box { width: 18px; height: 18px; border: 2px solid #d1d5db; border-radius: 5px; margin-right: 10px; transition: 0.2s; }
.custom-check input:checked + .box { background: #faa643; border-color: #faa643; position: relative; }
.custom-check input:checked + .box::after {
  content: ''; position: absolute; left: 5px; top: 1px; width: 4px; height: 8px;
  border: solid white; border-width: 0 2px 2px 0; transform: rotate(45deg);
}

.sidebar-footer { padding: 20px; border-top: 1px solid #f3f4f6; }
.update-btn {
  width: 100%; background: #faa643; color: #fff;
  border: none; padding: 14px; border-radius: 12px;
  font-weight: 700; cursor: pointer; transition: 0.2s;
  box-shadow: 0 4px 12px rgba(250, 166, 67, 0.2);
}
.update-btn:hover { transform: translateY(-1px); background: #d18c00; }

/* --- RESULTS AREA --- */
.results-area { flex: 1; padding: 32px; overflow-y: auto; background: #fff; }
.placeholder-state { text-align: center; margin-top: 15vh; color: #9ca3af; }
.pulse-icon { font-size: 3rem; margin-bottom: 1rem; }

.results-meta { margin-bottom: 24px; color: #6b7280; font-size: 0.9rem; }
.results-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}
.client-card {
  padding: 24px; border-radius: 16px; border: 1px solid #f3f4f6;
  background: #fff; transition: 0.3s;
}
.client-card:hover { border-color: #faa643; box-shadow: 0 10px 25px rgba(0,0,0,0.03); }

.card-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 16px; }
.avatar { 
  width: 44px; height: 44px; background: #fff4e5; color: #faa643;
  display: flex; align-items: center; justify-content: center;
  border-radius: 12px; font-weight: 700; font-size: 1.2rem;
}
.badge { font-size: 0.65rem; background: #ecfdf5; color: #10b981; padding: 4px 10px; border-radius: 20px; font-weight: 700; }
.account-type { font-size: 0.8rem; color: #9ca3af; margin: 4px 0 20px 0; }

.view-btn {
  width: 100%; background: #f9fafb; border: 1px solid #e5e7eb;
  padding: 8px; border-radius: 8px; font-weight: 600; cursor: pointer;
}

/* --- DARK MODE --- */
@media (prefers-color-scheme: dark) {
  .dashboard-wrapper { background: #111827; color: #f9fafb; }
  .top-nav { background: #1f2937; border-color: #374151; }
  .nav-title { color: #fff; }
  .nav-search-box { background: #374151; }
  .nav-search-box input { color: #fff; }
  .sidebar-right { background: #1f2937; border-color: #374151; }
  .results-area { background: #111827; }
  .client-card { background: #1f2937; border-color: #374151; }
  .header-label, .account-type { color: #9ca3af; }
  .box { border-color: #4b5563; }
}
</style>