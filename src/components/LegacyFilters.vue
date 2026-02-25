<template>
  <div class="mini-adv-sidebar">
    <header class="sidebar-header">
      <div class="brand">
        <span class="active-dot"></span>
        <h1>Advanced Filters</h1>
      </div>
      <button class="reset-link" @click="resetAll">Reset All</button>
    </header>

    <div class="global-search-wrapper">
      <input 
        v-model="globalSearch" 
        type="text" 
        placeholder="Search all settings..." 
        class="global-search-input"
      />
    </div>

    <div class="accordion-container">
      <details 
        :open="isSectionOpen('Identification')" 
        v-show="isSectionVisible('Identification')" 
        class="mini-card"
      >
        <summary><span class="accordion-arrow"></span>Identification</summary>
        <div class="card-inner">
          <div class="field">
            <label>Accounting IDs</label>
            <div class="mini-tag-box">
              <span v-for="(tag, i) in filters.accountingIds" :key="i" class="mini-tag">
                {{ tag }} <b @click="removeTag('accountingIds', i)">×</b>
              </span>
              <input @keydown.enter="addTag($event, 'accountingIds')" placeholder="Add ID..." />
            </div>
          </div>
          <div class="field">
            <label>Cargo Types</label>
            <div class="mini-tag-box">
              <span v-for="(tag, i) in filters.cargoTypes" :key="i" class="mini-tag">
                {{ tag }} <b @click="removeTag('cargoTypes', i)">×</b>
              </span>
              <input @keydown.enter="addTag($event, 'cargoTypes')" placeholder="Add Type..." />
            </div>
          </div>
        </div>
      </details>

      <details 
        :open="isSectionOpen('Documents & Comms')" 
        v-show="isSectionVisible('Documents & Comms')" 
        class="mini-card"
      >
        <summary><span class="accordion-arrow"></span>Documents & Comms</summary>
        <div class="card-inner">
          <div class="field">
            <label>Attached Docs ({{ filters.docTypes.length }})</label>
            <div class="mini-search-list">
              <input v-model="search.docs" placeholder="Search 45+ docs..." class="inner-search" />
              <div class="scroll-box">
                <div 
                  v-for="opt in filteredDocs" :key="opt.value" 
                  class="row" :class="{ selected: filters.docTypes.includes(opt.value) }"
                  @click="toggleSelection('docTypes', opt.value)"
                >
                  <input type="checkbox" :checked="filters.docTypes.includes(opt.value)" />
                  {{ opt.label }}
                </div>
              </div>
            </div>
          </div>
          <div class="field">
            <label>Auto Send BOL</label>
            <div class="mini-segmented">
              <button 
                v-for="opt in autoSendOptions" :key="opt.value"
                :class="{ active: filters.autoSend.includes(opt.value) }"
                @click="toggleSelection('autoSend', opt.value)"
              >{{ opt.label }}</button>
            </div>
          </div>
        </div>
      </details>

      <details 
        :open="isSectionOpen('Business & Ownership')" 
        v-show="isSectionVisible('Business & Ownership')" 
        class="mini-card"
      >
        <summary><span class="accordion-arrow"></span>Business & Ownership</summary>
        <div class="card-inner">
          <div class="field">
            <label>Business Type/s ({{ filters.bizTypes.length }})</label>
            <div class="mini-search-list">
              <input v-model="search.biz" placeholder="Search 30+ types..." class="inner-search" />
              <div class="scroll-box">
                <div 
                  v-for="opt in filteredBiz" :key="opt.value" 
                  class="row" :class="{ selected: filters.bizTypes.includes(opt.value) }"
                  @click="toggleSelection('bizTypes', opt.value)"
                >
                  <input type="checkbox" :checked="filters.bizTypes.includes(opt.value)" />
                  {{ opt.label }}
                </div>
              </div>
            </div>
          </div>
          <div class="field">
            <label>Client Created By</label>
            <select v-model="filters.createdBy" multiple class="mini-select">
              <option v-for="u in users" :key="u.value" :value="u.value">{{ u.label }}</option>
            </select>
          </div>
        </div>
      </details>
    </div>

    <footer class="sidebar-footer">
      <button class="btn-apply" @click="applyFilters">Apply Configuration</button>
    </footer>
  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue'

// --- SEARCH STATE ---
const globalSearch = ref('')

// --- DATA ---
const docOptions = [{ value: "[NONE]", label: "Unassigned" }, { value: "94", label: "BOL 01" }, { value: "95", label: "BOL 05" }, { value: "6", label: "Bid" }, { value: "1", label: "Bill of Lading" }, { value: "11", label: "BoL Cancellation" }, { value: "58", label: "CSA" }, { value: "93", label: "Carrier Agreement Amendment" }, { value: "29", label: "Claims Form" }, { value: "100", label: "Client Approval" }, { value: "2", label: "Consignment" }, { value: "27", label: "Credit Application" }, { value: "28", label: "Credit Form" }, { value: "54", label: "Customs Clearance" }, { value: "92", label: "E-Signed Carrier Agreement" }, { value: "104", label: "EPA Form" }, { value: "103", label: "Export Documents" }, { value: "96", label: "External TMS BOL" }, { value: "56", label: "Gate Pass" }, { value: "102", label: "Import Documents" }, { value: "4", label: "Inspection" }, { value: "26", label: "Inspection Image" }, { value: "25", label: "Inspection Video" }, { value: "8", label: "Insurance" }, { value: "3", label: "Invoice" }, { value: "101", label: "Ocean BOL" }, { value: "99", label: "Order For Service" }, { value: "7", label: "PU Fax Reply" }, { value: "10", label: "Pickup Notification" }, { value: "59", label: "Pre-Load Condition Report" }, { value: "55", label: "Rating" }, { value: "9", label: "Registration" }, { value: "5", label: "Release" }, { value: "30", label: "Reporting Estimate" }, { value: "31", label: "Supporting Doc" }, { value: "98", label: "Terms of Service" }, { value: "105", label: "UDA Client Document" }, { value: "97", label: "UDA Driver Document" }, { value: "53", label: "Unknown Document" }, { value: "106", label: "Vendor Bills" }, { value: "57", label: "W9" }, { value: "45", label: "billOfLading" }, { value: "48", label: "factorInvoice" }, { value: "46", label: "invoice" }, { value: "49", label: "laborReceipt" }, { value: "50", label: "noticeOfAssignment" }, { value: "51", label: "other" }, { value: "47", label: "rateConfirmation" }, { value: "52", label: "weightTicket" }]
const bizOptions = [{ value: "[NONE]", label: "Unassigned" }, { value: "AUCTION", label: "Auction" }, { value: "SEA", label: "Auto Service" }, { value: "BANK", label: "Bank" }, { value: "BODY SHOP", label: "Body Shop" }, { value: "BROKER", label: "Broker" }, { value: "BUSINESS", label: "Business" }, { value: "CARRIER", label: "Carrier" }, { value: "DEALER", label: "Dealer" }, { value: "FLEET", label: "Fleet" }, { value: "HOTEL", label: "Hotel" }, { value: "IN HOUSE TRANSPORT", label: "In House Transport" }, { value: "INSPECTION", label: "Inspection" }, { value: "INSPECTIONCENTER", label: "Inspection Center" }, { value: "INSURANCE CO.", label: "Insurance Co." }, { value: "MANUFACTURER", label: "Manufacturer" }, { value: "MARSHALLING", label: "Marshalling" }, { value: "NVOCC", label: "NVOCC" }, { value: "OEM", label: "OEM" }, { value: "OTHER", label: "Other" }, { value: "CAPTIVE", label: "Private Party" }, { value: "RAIL", label: "Rail" }, { value: "RENTAL", label: "Rental" }, { value: "REPO AGENT", label: "Repo Agent" }, { value: "REPO YARD", label: "Repo Yard" }, { value: "INDIVIDUAL", label: "Residence" }, { value: "AUTO_PROCESSING", label: "Seasonal Transport" }, { value: "STORAGE YARD", label: "Storage Yard" }, { value: "TOW YARD", label: "Tow Yard" }, { value: "RORO", label: "Transport" }, { value: "VDC", label: "Vehicle Distribution Center" }, { value: "VPC", label: "Vehicle Processing Center" }]
const users = [{ value: "25149", label: "ALFRED OBERMANN" }, { value: "25153", label: "AMELIA PARENT" }, { value: "25140", label: "CARGOTEL ADMIN" }, { value: "25354", label: "CGT API" }, { value: "25201", label: "DEBBIE K ROGDOWSKI" }, { value: "25144", label: "DIANE ROGDOWSKI" }, { value: "25154", label: "DONALD CALEFSKY" }, { value: "25147", label: "EDGAR REYES COLLADONA" }, { value: "25151", label: "JAMES GRECONA" }, { value: "25155", label: "JEFFERSON HILLMAN" }, { value: "25146", label: "JONAH ZACHARY" }, { value: "25142", label: "JONATHAN LOUISIANA" }, { value: "25189", label: "LAWRENCE BODEROSKY" }, { value: "25152", label: "SOPHIA DOMINICI" }]
const autoSendOptions = [{ value: "[NONE]", label: "None" }, { value: "E", label: "Email" }, { value: "F", label: "Fax" }]

// --- STATE ---
const filters = reactive({ accountingIds: [], cargoTypes: [], docTypes: [], bizTypes: [], autoSend: [], createdBy: [] })
const search = reactive({ docs: '', biz: '' })

// --- FILTERING LOGIC ---
const searchableKeywords = {
  'Identification': 'accounting ids cargo types identification labels tags',
  'Documents & Comms': 'attached docs auto send bol email fax documents communications comms',
  'Business & Ownership': 'business types client created by ownership users'
}

const isSectionVisible = (title) => {
  if (!globalSearch.value) return true;
  const term = globalSearch.value.toLowerCase();
  return title.toLowerCase().includes(term) || (searchableKeywords[title] || '').includes(term);
}

const isSectionOpen = (title) => {
  // Always open "Identification" by default on load, 
  // or open any section that matches a search term
  if (!globalSearch.value) return title === 'Identification';
  return isSectionVisible(title);
}

const filteredDocs = computed(() => docOptions.filter(o => o.label.toLowerCase().includes(search.docs.toLowerCase())))
const filteredBiz = computed(() => bizOptions.filter(o => o.label.toLowerCase().includes(search.biz.toLowerCase())))

// --- ACTIONS ---
const addTag = (e, key) => {
  const v = e.target.value.trim();
  if (v && !filters[key].includes(v)) { filters[key].push(v); e.target.value = ''; }
}
const removeTag = (key, i) => filters[key].splice(i, 1)
const toggleSelection = (key, val) => {
  const i = filters[key].indexOf(val);
  i > -1 ? filters[key].splice(i, 1) : filters[key].push(val);
}
const resetAll = () => {
  Object.keys(filters).forEach(k => filters[k] = [])
  search.docs = ''; search.biz = ''; globalSearch.value = '';
}
const applyFilters = () => console.log("Final:", JSON.parse(JSON.stringify(filters)))
</script>

<style scoped>
.mini-adv-sidebar {
  background: #fff; border: 1px solid #e2e8f0; border-radius: 8px;
  display: flex; flex-direction: column; font-family: 'Inter', sans-serif;
}
.sidebar-header {
  padding: 12px 16px; display: flex; justify-content: space-between; align-items: center;
  border-bottom: 1px solid #f1f5f9;
}
.brand { display: flex; align-items: center; gap: 8px; }
.active-dot { width: 6px; height: 6px; background: #7c4dff; border-radius: 50%; }
.brand h1 { font-size: 0.85rem; font-weight: 700; color: #1e293b; margin: 0; }
.reset-link { background: none; border: none; color: #64748b; font-size: 0.7rem; font-weight: 600; cursor: pointer; }

.global-search-wrapper { padding: 10px 16px; background: #fff; border-bottom: 1px solid #f1f5f9; }
.global-search-input {
  width: 100%; padding: 8px 12px; border: 1px solid #e2e8f0; border-radius: 6px;
  font-size: 0.75rem; background: #fcfcfc; outline: none; transition: all 0.2s ease;
  max-width: 200px;
}
.global-search-input:focus { border-color: #7c4dff; background: #fff; box-shadow: 0 0 0 3px rgba(124, 77, 255, 0.1); }

.accordion-container { max-height: 70vh; overflow-y: auto; }
.mini-card { border-bottom: 1px solid #f1f5f9; }
.mini-card summary {
  padding: 10px 16px; font-size: 0.75rem; font-weight: 700; color: #475569;
  background: #f8fafc; cursor: pointer; outline: none; list-style: none;
}
.card-inner { padding: 12px 16px; }

.field { margin-bottom: 12px; }
.field label { display: block; font-size: 0.7rem; font-weight: 600; color: #94a3b8; margin-bottom: 4px; text-transform: uppercase; }

.mini-tag-box { border: 1px solid #e2e8f0; border-radius: 4px; padding: 4px; display: flex; flex-wrap: wrap; gap: 4px; min-height: 40px; }
.mini-tag { background: #ede7f6; color: #7c4dff; font-size: 0.65rem; padding: 1px 6px; border-radius: 3px; display: flex; align-items: center; gap: 4px; }
.mini-tag b { cursor: pointer; font-style: normal; }
.mini-tag-box input { border: none; outline: none; font-size: 0.75rem; flex: 1; min-width: 50px; }

.mini-search-list { border: 1px solid #e2e8f0; border-radius: 4px; overflow: hidden; }
.inner-search { width: 100%; border: none; border-bottom: 1px solid #e2e8f0; padding: 5px 8px; font-size: 0.75rem; background: #f8fafc; }
.scroll-box { height: 120px; overflow-y: auto; }
.row { padding: 4px 8px; font-size: 0.75rem; display: flex; align-items: center; gap: 6px; cursor: pointer; }
.row:hover { background: #f3f0ff; }
.row.selected { background: #ede7f6; color: #7c4dff; font-weight: 600; }
.row input { width: unset; }

.mini-segmented { display: flex; background: #f1f5f9; padding: 2px; border-radius: 4px; }
.mini-segmented button { flex: 1; border: none; background: transparent; padding: 4px; font-size: 0.7rem; border-radius: 3px; cursor: pointer; color: #64748b; }
.mini-segmented button.active { background: #fff; color: #7c4dff; font-weight: 700; box-shadow: 0 1px 2px rgba(0,0,0,0.1); }

.mini-select { width: 100%; height: 80px; border: 1px solid #e2e8f0; border-radius: 4px; font-size: 0.75rem; }

.sidebar-footer { padding: 12px 16px; border-top: 1px solid #e2e8f0; }
.btn-apply { width: 100%; background: #7c4dff; color: #fff; border: none; padding: 8px; border-radius: 6px; font-weight: 700; font-size: 0.8rem; cursor: pointer; }
/* Accordion arrow styles */
.accordion-arrow {
  display: inline-block;
  width: 10px;
  height: 10px;
  margin-right: 8px;
  border-right: 2px solid #7c4dff;
  border-bottom: 2px solid #7c4dff;
  transform: rotate(45deg);
  transition: transform 0.2s;
  position: relative;
  left: 95%;
}
details[open] > summary .accordion-arrow {
  transform: rotate(-135deg);
}
</style>
