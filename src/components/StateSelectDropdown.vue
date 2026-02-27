<template>
  <div class="state-select-heading">State Select</div>
  <div class="filter-pill" :class="{ 'is-active': selectedStates.length > 0 }" v-click-outside="close">
    <span class="pill-label">States</span>
    
    <div class="pill-trigger" @click="toggleDropdown">
      <span v-if="selectedStates.length === 0" class="pill-placeholder">All States</span>
      <span v-else class="pill-value">
        <template v-for="(abbr, idx) in selectedStates" :key="abbr">
          {{ getStateName(abbr) }}
          <span class="remove-btn" @click.stop="removeState(abbr)">&times;</span>
          <span v-if="idx < selectedStates.length - 1">, </span>
        </template>
      </span>
      <span class="pill-chevron" :class="{ 'is-open': dropdownOpen }"></span>
    </div>

    <div v-if="dropdownOpen" class="state-dropdown">
      <div class="dropdown-header">
        <button @click="selectedStates = []" class="text-btn">Clear All</button>
      </div>
      <div style="padding: 8px 0 12px 0;">
        <div class="chip-list" v-if="selectedStates.length">
          <span v-for="abbr in selectedStates" :key="abbr" class="chip">
            {{ getStateName(abbr) }}
            <span class="chip-remove" @click.stop="removeState(abbr)">&times;</span>
          </span>
        </div>
        <input
          type="text"
          v-model="searchState"
          placeholder="Search state/province..."
          class="inner-search adv-search"
          style="width:100%; font-size:13px; max-width: 250px; margin-bottom: 8px;"
        />
        <div class="options-grid">
          <div class="group-title" v-if="filteredUsaStates.length">USA States</div>
          <div v-for="state in filteredUsaStates" :key="state.abbr" class="option-item" :class="{selected: selectedStates.includes(state.abbr)}" @click="!selectedStates.includes(state.abbr) && addSelectedStateFromList(state.abbr)">
            <span class="option-name">{{ state.name }}</span>
          </div>
          <div class="group-title" v-if="filteredCanadaProvinces.length">Canada Provinces</div>
          <div v-for="province in filteredCanadaProvinces" :key="province.abbr" class="option-item" :class="{selected: selectedStates.includes(province.abbr)}" @click="!selectedStates.includes(province.abbr) && addSelectedStateFromList(province.abbr)">
            <span class="option-name">{{ province.name }}</span>
          </div>
        </div>

      </div>
      <!-- No scroll-area, all handled in select above -->
      <div v-if="!filteredUsaStates.length && !filteredCanadaProvinces.length" class="no-results">No results found.</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';


const selectedStates = ref([]);
const dropdownOpen = ref(false);

const selectStateValue = ref(""); // No longer used for custom list
function addSelectedStateFromList(abbr) {
  if (abbr && !selectedStates.value.includes(abbr)) {
    selectedStates.value.push(abbr);
  }
}
const searchState = ref("");

const filteredUsaStates = computed(() => {
  if (!searchState.value) return usaStates;
  return usaStates.filter(s =>
    s.name.toLowerCase().includes(searchState.value.toLowerCase()) ||
    s.abbr.toLowerCase().includes(searchState.value.toLowerCase())
  );
});
const filteredCanadaProvinces = computed(() => {
  if (!searchState.value) return canadaProvinces;
  return canadaProvinces.filter(p =>
    p.name.toLowerCase().includes(searchState.value.toLowerCase()) ||
    p.abbr.toLowerCase().includes(searchState.value.toLowerCase())
  );
});

function addSelectedState() {
  const abbr = selectStateValue.value;
  if (abbr && !selectedStates.value.includes(abbr)) {
    selectedStates.value.push(abbr);
  }
  selectStateValue.value = "";
}

function selectAllFiltered() {
  const all = [
    ...filteredUsaStates.value.map(s => s.abbr),
    ...filteredCanadaProvinces.value.map(p => p.abbr)
  ];
  selectedStates.value = Array.from(new Set([...selectedStates.value, ...all]));
}

function deselectAllFiltered() {
  const filtered = [
    ...filteredUsaStates.value.map(s => s.abbr),
    ...filteredCanadaProvinces.value.map(p => p.abbr)
  ];
  selectedStates.value = selectedStates.value.filter(abbr => !filtered.includes(abbr));
}

const usaStates = [
  { name: 'Alabama', abbr: 'AL' }, { name: 'Alaska', abbr: 'AK' }, { name: 'Arizona', abbr: 'AZ' }, { name: 'Arkansas', abbr: 'AR' },
  { name: 'California', abbr: 'CA' }, { name: 'Colorado', abbr: 'CO' }, { name: 'Connecticut', abbr: 'CT' }, { name: 'Delaware', abbr: 'DE' },
  { name: 'Florida', abbr: 'FL' }, { name: 'Georgia', abbr: 'GA' }, { name: 'Hawaii', abbr: 'HI' }, { name: 'Idaho', abbr: 'ID' },
  { name: 'Illinois', abbr: 'IL' }, { name: 'Indiana', abbr: 'IN' }, { name: 'Iowa', abbr: 'IA' }, { name: 'Kansas', abbr: 'KS' },
  { name: 'Kentucky', abbr: 'KY' }, { name: 'Louisiana', abbr: 'LA' }, { name: 'Maine', abbr: 'ME' }, { name: 'Maryland', abbr: 'MD' },
  { name: 'Massachusetts', abbr: 'MA' }, { name: 'Michigan', abbr: 'MI' }, { name: 'Minnesota', abbr: 'MN' }, { name: 'Mississippi', abbr: 'MS' },
  { name: 'Missouri', abbr: 'MO' }, { name: 'Montana', abbr: 'MT' }, { name: 'Nebraska', abbr: 'NE' }, { name: 'Nevada', abbr: 'NV' },
  { name: 'New Hampshire', abbr: 'NH' }, { name: 'New Jersey', abbr: 'NJ' }, { name: 'New Mexico', abbr: 'NM' }, { name: 'New York', abbr: 'NY' },
  { name: 'North Carolina', abbr: 'NC' }, { name: 'North Dakota', abbr: 'ND' }, { name: 'Ohio', abbr: 'OH' }, { name: 'Oklahoma', abbr: 'OK' },
  { name: 'Oregon', abbr: 'OR' }, { name: 'Pennsylvania', abbr: 'PA' }, { name: 'Rhode Island', abbr: 'RI' }, { name: 'South Carolina', abbr: 'SC' },
  { name: 'South Dakota', abbr: 'SD' }, { name: 'Tennessee', abbr: 'TN' }, { name: 'Texas', abbr: 'TX' }, { name: 'Utah', abbr: 'UT' },
  { name: 'Vermont', abbr: 'VT' }, { name: 'Virginia', abbr: 'VA' }, { name: 'Washington', abbr: 'WA' }, { name: 'West Virginia', abbr: 'WV' },
  { name: 'Wisconsin', abbr: 'WI' }, { name: 'Wyoming', abbr: 'WY' }
];

const canadaProvinces = [
  { name: 'Alberta', abbr: 'AB' }, { name: 'British Columbia', abbr: 'BC' }, { name: 'Manitoba', abbr: 'MB' },
  { name: 'New Brunswick', abbr: 'NB' }, { name: 'Newfoundland and Labrador', abbr: 'NL' }, { name: 'Nova Scotia', abbr: 'NS' },
  { name: 'Ontario', abbr: 'ON' }, { name: 'Prince Edward Island', abbr: 'PE' }, { name: 'Quebec', abbr: 'QC' },
  { name: 'Saskatchewan', abbr: 'SK' }, { name: 'Northwest Territories', abbr: 'NT' }, { name: 'Nunavut', abbr: 'NU' }, { name: 'Yukon', abbr: 'YT' }
];

const toggleDropdown = () => dropdownOpen.value = !dropdownOpen.value;
const close = () => dropdownOpen.value = false;

const removeState = (abbr) => {
  const idx = selectedStates.value.indexOf(abbr);
  if (idx > -1) selectedStates.value.splice(idx, 1);
};

const getStateName = (abbr) => {
  const all = [...usaStates, ...canadaProvinces];
  return all.find(s => s.abbr === abbr)?.name || abbr;
};

// Simple Click Outside Directive Logic
const vClickOutside = {
  mounted(el, binding) {
    el.clickOutsideEvent = (event) => {
      if (!(el === event.target || el.contains(event.target))) {
        binding.value();
      }
    };
    document.addEventListener("click", el.clickOutsideEvent);
  },
  unmounted(el) {
    document.removeEventListener("click", el.clickOutsideEvent);
  },
};

// Register directive for script setup
defineExpose({ vClickOutside });
</script>

<style scoped>
/* Reuse your existing Filter Pill styles for consistency */
.filter-pill {
  position: relative;
  display: inline-flex;
  align-items: center;
  background: #fff8ef;
  border: 1.5px solid #faa643;
  border-radius: 8px;
  padding: 6px 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-top: 20px;}

.filter-pill.is-active {
  border-color: #faa643;
  background: #fff8ef;
}

.pill-label {
  font-size: 11px;
  font-weight: 800;
  color: #94a3b8;
  text-transform: uppercase;
  margin-right: 10px;
  padding-right: 10px;
  border-right: 1px solid #faa643;
}

 .is-active .pill-label { color: #faa643; border-right-color: #faa643; }

.pill-trigger {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 100px;
}

.pill-value {
  font-size: 13px;
  font-weight: 600;
  color: #1e293b;
}
.option-item input[type="checkbox"] {
  width: unset;
}

.pill-count {
  background: #faa643;
  color: #fff;
  padding: 1px 6px;
  border-radius: 10px;
  font-size: 10px;
}

/* Dropdown Container */
.state-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  max-height: 400px;
  overflow: auto;
  left: 0;
  width: 450px; /* Wider to allow grid layout */
  background: #fff8ef;
  border: 1.5px solid #faa643;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
  z-index: 2000;
  padding: 12px;
}

.dropdown-header {
  padding-bottom: 8px;
  border-bottom: 1.5px solid #faa643;
  margin-bottom: 10px;
  display: flex;
  justify-content: flex-end;
}

.text-btn {
  background: none;
  border: none;
  color: #faa643;
  font-size: 11px;
  font-weight: 700;
  cursor: pointer;
}

.scroll-area {
  max-height: 300px;
  overflow-y: auto;
}

.group-title {
  font-size: 11px;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
  margin: 12px 0 8px 0;
}

.options-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 3 Columns for compact view */
  gap: 4px;
}

.option-item {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px;
  cursor: pointer;
  border-radius: 4px;
  transition: background 0.1s;
}

.option-item.selected {
  background: #faa643;
}

.option-item:hover { background: #faa643; }
.chip-list {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 6px;
}
.chip {
  color: #fff8ef;
  background: #faa643;
  border-radius: 12px;
  padding: 2px 10px 2px 8px;
  font-size: 12px;
  display: flex;
  align-items: center;
}
.chip-remove {
  margin-left: 4px;
  color: rgb(167, 34, 34);
  font-weight: bold;
  cursor: pointer;
  font-size: 13px;
}
.chip-remove:hover {
  color: #d18c00;
}

.adv-actions {
  display: flex;
  gap: 8px;
  margin-bottom: 4px;
}
.adv-btn {
  background: #ede9fe;
  color: #7c4dff;
  border: none;
  border-radius: 6px;
  padding: 2px 10px;
  font-size: 11px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.15s;
}
.adv-btn:hover {
  background: #c7d2fe;
}

.no-results {
  color: #b91c1c;
  font-size: 13px;
  text-align: center;
  margin-top: 12px;
}

mark {
  background: #fef08a;
  color: #92400e;
  border-radius: 2px;
  padding: 0 2px;
}

.option-name {
  font-size: 12px;
  color: #334155;
  white-space: nowrap;
}

input[type="checkbox"] { accent-color: #7c4dff; }

.pill-chevron {
  width: 6px;
  height: 6px;
  border-right: 2px solid #94a3b8;
  border-bottom: 2px solid #94a3b8;
  transform: rotate(45deg);
  transition: transform 0.2s;
}

.pill-chevron.is-open { transform: rotate(-135deg) translateY(-2px); }

@media (max-width: 600px) {
  .state-dropdown { width: 90vw; }
  .options-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (prefers-color-scheme: dark) {
  .filter-pill {
    background: #23263a;
    border-color: #353a50;
    color: #e0e6f5;
  }
  .filter-pill.is-active {
    border-color: #a78bfa;
    background: #23204a;
  }
  .pill-label {
    color: #a3aed6;
    border-right-color: #353a50;
  }
  .is-active .pill-label {
    color: #a78bfa;
    border-right-color: #a78bfa;
  }
  .pill-value {
    color: #e0e6f5;
  }
  .pill-placeholder {
    color: #64748b;
  }
  .pill-chevron {
    border-right-color: #a3aed6;
    border-bottom-color: #a3aed6;
  }
  .pill-chevron.is-open {
    border-right-color: #a78bfa;
    border-bottom-color: #a78bfa;
  }
  .state-dropdown {
    background: #23263a;
    border-color: #a78bfa;
    color: #e0e6f5;
    box-shadow: 0 10px 25px rgba(0,0,0,0.7);
  }
  .dropdown-header {
    border-bottom-color: #353a50;
  }
  .chip {
    background: #e9d6ba;
    color: #23204a;
    border: 1px solid #faa643;
  }
  .chip-remove {
    color: #faa643;
  }
  .chip-remove:hover {
    color: #d18c00;
  }
  .group-title {
    color: #faa643;
    font-weight: 700;
    letter-spacing: 0.5px;
  }
  .option-item {
    background: transparent;
    color: #e0e6f5;
  }
  .option-item.selected {
    background: #faa643;
    color: #fff;
    border-radius: 6px;
  }
  .option-item:hover {
    background: #fff8ef;
    color: #faa643;
    border-radius: 6px;
  }
  .option-name {
    color: #e0e6f5;
  }
  .no-results {
    color: #faa643;
  }
  .text-btn {
    color: #faa643;
  }
  .inner-search.adv-search {
    background: #23263a;
    color: #e0e6f5;
    border-bottom: 1.5px solid #faa643;
  }
}

.remove-btn {
  color: #ef4444;
  font-weight: bold;
  margin-left: 2px;
  cursor: pointer;
  font-size: 13px;
}
.remove-btn:hover { color: #b91c1c; }
</style>
<style scoped>
.state-select-heading {
  font-size: 1.4rem;
  font-weight: 700;
  color: #147393;
  margin-bottom: 0.5rem;
  margin-top: 0.5rem;
  letter-spacing: 0.01em;
  font-family: 'Inter', -apple-system, sans-serif;
}
</style>