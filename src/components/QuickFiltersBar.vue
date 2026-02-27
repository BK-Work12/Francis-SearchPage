<template>
 
  <div v-if="restrictTo" class="mini-filter-system">
     <div class="quick-filters-heading">Quick Filters</div>
    <div class="layout">
      <div class="first">
        <header class="mini-header">
          <div class="brand">
            <div class="dot-indicator"></div>
            <span class="title">Global Filters</span>
            <span class="count-badge">{{ activeFilterCount }} Active</span>
          </div>
          <div class="header-actions">
            <button class="text-btn" @click="resetAll">Reset</button>
            <button class="apply-btn" @click="onSearch">Apply Search</button>
          </div>
        </header>

        <div class="main-search-container">
          <input 
            v-model="globalSearch" 
            type="text" 
            placeholder="Search all filters" 
            class="global-search-input"
          />
        </div>

        <div class="accordion-container">
          <details :open="shouldOpen('Identification & Accounting')" class="filter-section" v-show="isSectionVisible('Identification & Accounting')">
            <summary><span class="accordion-arrow"></span>Identification & Accounting</summary>
            <div class="section-content">
              <div class="field-row">
                <div class="mini-tag-input">
                  <label>Accounting IDs</label>
                  <div class="tag-wrap">
                    <span v-for="(id, i) in filters.accountingIds" :key="i" class="mini-tag">
                      {{ id }} <i @click="removeTag('accountingIds', i)">×</i>
                    </span>
                    <input @keydown.enter="addTag($event, 'accountingIds')" placeholder="Add ID..." />
                  </div>
                </div>
              </div>
              <div class="field-row">
                <div class="mini-tag-input">
                  <label>Cargo Types</label>
                  <div class="tag-wrap">
                    <span v-for="(ct, i) in filters.cargoTypes" :key="i" class="mini-tag">
                      {{ ct }} <i @click="removeTag('cargoTypes', i)">×</i>
                    </span>
                    <input @keydown.enter="addTag($event, 'cargoTypes')" placeholder="Add Type..." />
                  </div>
                </div>
              </div>
              <div class="field-row">
                <label>Created By</label>
                <select v-model="filters.clientCreatedBy" multiple class="compact-multi">
                  <option value="[NONE]">Unassigned</option>
                  <option v-for="u in users" :key="u.val" :value="u.val">{{ u.label }}</option>
                </select>
              </div>
            </div>
          </details>

          <details :open="shouldOpen('Account Status & Flags')" class="filter-section" v-show="isSectionVisible('Account Status & Flags')">
            <summary><span class="accordion-arrow"></span>Account Status & Flags</summary>
            <div class="section-content grid-2-col">
              <div v-for="(label, key) in statusToggles" :key="key" class="mini-toggle-box">
                <span class="tiny-label">{{ label }}</span>
                <div class="micro-tri-state">
                  <button :class="{active: filters[key] === null}" @click="filters[key] = null">Any</button>
                  <button :class="{active: filters[key] === false}" @click="filters[key] = false">No</button>
                  <button :class="{active: filters[key] === true}" @click="filters[key] = true">Yes</button>
                </div>
              </div>
            </div>
          </details>

          <details :open="shouldOpen('Documents & Business Types')" class="filter-section" v-show="isSectionVisible('Documents & Business Types')">
            <summary><span class="accordion-arrow"></span>Documents & Business Types</summary>
            <div class="section-content">
              <div class="field-row">
                <label>Attached Docs ({{ filters.attachedDocs.length }})</label>
                <div class="list-container">
                  <input v-model="search.docs" placeholder="Search 45+ docs..." class="inner-search" />
                  <div class="micro-scroll">
                    <div v-for="doc in filteredDocs" :key="doc.val" class="list-row" @click="toggleSelection('attachedDocs', doc.val)">
                      <input type="checkbox" :checked="filters.attachedDocs.includes(doc.val)" /> {{ doc.label }}
                    </div>
                  </div>
                </div>
              </div>
              <div class="field-row">
                <label>Business Types ({{ filters.businessTypes.length }})</label>
                <div class="list-container">
                  <input v-model="search.biz" placeholder="Search 30+ types..." class="inner-search" />
                  <div class="micro-scroll">
                    <div v-for="biz in filteredBiz" :key="biz.val" class="list-row" @click="toggleSelection('businessTypes', biz.val)">
                      <input type="checkbox" :checked="filters.businessTypes.includes(biz.val)" /> {{ biz.label }}
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </details>

          <details :open="shouldOpen('Carrier & Domains')" class="filter-section" v-show="isSectionVisible('Carrier & Domains')">
            <summary><span class="accordion-arrow"></span>Carrier & Domains</summary>
            <div class="section-content">
              <div class="field-row">
                <label>Auto-Send BOL</label>
                <div class="inline-checks">
                  <label v-for="opt in autoSendOptions" :key="opt.val">
                    <input type="checkbox" :value="opt.val" v-model="filters.autoSendBol" /> {{ opt.label }}
                  </label>
                </div>
              </div>
              <div class="field-row">
                <label>Carrier Cargo Types</label>
                <textarea v-model="filters.carrierCargoText" class="mini-area" rows="1"></textarea>
              </div>
              <div class="field-row">
                <label>Only Cargo Types</label>
                <select v-model="filters.containsOnlyCargo" multiple class="compact-multi-small">
                  <option v-for="opt in cargoList" :key="opt" :value="opt">{{ opt }}</option>
                </select>
              </div>
              <div class="field-row">
                <label>Central Dispatch Exclusions</label>
                <textarea v-model="filters.dispatchExclude" class="mini-area" rows="1"></textarea>
              </div>
              <div class="field-row">
                <label>Business Domains</label>
                <select v-model="filters.businessDomains" multiple class="compact-multi-small">
                  <option value="513069">Demo Transport</option>
                  <option value="517797">Ford Motor Company</option>
                </select>
              </div>
            </div>
          </details>
        </div>
      </div>

      <div class="legacy-wrapper">
        <LegacyFilters />
      </div>
      <div class="third">
        <LegacyLocationFilters />
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue'
import LegacyLocationFilters from './LegacyLocationFilters.vue'
import LegacyFilters from './LegacyFilters.vue'

const props = defineProps({ restrictTo: Boolean })

// --- GLOBAL SEARCH STATE ---
const globalSearch = ref('')

// --- DATA PRESERVED FROM ORIGINAL ---
const users = [{ val: "25149", label: "ALFRED OBERMANN" }, { val: "25153", label: "AMELIA PARENT" }, { val: "25140", label: "CARGOTEL ADMIN" }, { val: "25354", label: "CGT API" }, { val: "25201", label: "DEBBIE K ROGDOWSKI" }, { val: "25144", label: "DIANE ROGDOWSKI" }, { val: "25154", label: "DONALD CALEFSKY" }, { val: "25147", label: "EDGAR REYES COLLADONA" }, { val: "25151", label: "JAMES GRECONA" }, { val: "25155", label: "JEFFERSON HILLMAN" }, { val: "25146", label: "JONAH ZACHARY" }, { val: "25142", label: "JONATHAN LOUISIANA" }, { val: "25189", label: "LAWRENCE BODEROSKY" }, { val: "25152", label: "SOPHIA DOMINICI" }]
const docOptions = [{ val: "[NONE]", label: "Unassigned" }, { val: "94", label: "BOL 01" }, { val: "95", label: "BOL 05" }, { val: "6", label: "Bid" }, { val: "1", label: "Bill of Lading" }, { val: "11", label: "BoL Cancellation" }, { val: "58", label: "CSA" }, { val: "93", label: "Carrier Agreement Amendment" }, { val: "29", label: "Claims Form" }, { val: "100", label: "Client Approval" }, { val: "2", label: "Consignment" }, { val: "27", label: "Credit Application" }, { val: "28", label: "Credit Form" }, { val: "54", label: "Customs Clearance" }, { val: "92", label: "E-Signed Carrier Agreement" }, { val: "104", label: "EPA Form" }, { val: "103", label: "Export Documents" }, { val: "96", label: "External TMS BOL" }, { val: "56", label: "Gate Pass" }, { val: "102", label: "Import Documents" }, { val: "4", label: "Inspection" }, { val: "26", label: "Inspection Image" }, { val: "25", label: "Inspection Video" }, { val: "8", label: "Insurance" }, { val: "3", label: "Invoice" }, { val: "101", label: "Ocean BOL" }, { val: "99", label: "Order For Service" }, { val: "7", label: "PU Fax Reply" }, { val: "10", label: "Pickup Notification" }, { val: "59", label: "Pre-Load Condition Report" }, { val: "55", label: "Rating" }, { val: "9", label: "Registration" }, { val: "5", label: "Release" }, { val: "30", label: "Reporting Estimate" }, { val: "31", label: "Supporting Doc" }, { val: "98", label: "Terms of Service" }, { val: "105", label: "UDA Client Document" }, { val: "97", label: "UDA Driver Document" }, { val: "53", label: "Unknown Document" }, { val: "106", label: "Vendor Bills" }, { val: "57", label: "W9" }, { val: "45", label: "billOfLading" }, { val: "48", label: "factorInvoice" }, { val: "46", label: "invoice" }, { val: "49", label: "laborReceipt" }, { val: "50", label: "noticeOfAssignment" }, { val: "51", label: "other" }, { val: "47", label: "rateConfirmation" }, { val: "52", label: "weightTicket" }]
const bizOptions = [{ val: "[NONE]", label: "Unassigned" }, { val: "AUCTION", label: "Auction" }, { val: "SEA", label: "Auto Service" }, { val: "BANK", label: "Bank" }, { val: "BODY SHOP", label: "Body Shop" }, { val: "BROKER", label: "Broker" }, { val: "BUSINESS", label: "Business" }, { val: "CARRIER", label: "Carrier" }, { val: "DEALER", label: "Dealer" }, { val: "FLEET", label: "Fleet" }, { val: "HOTEL", label: "Hotel" }, { val: "IN HOUSE TRANSPORT", label: "In House Transport" }, { val: "INSPECTION", label: "Inspection" }, { val: "INSPECTIONCENTER", label: "Inspection Center" }, { val: "INSURANCE CO.", label: "Insurance Co." }, { val: "MANUFACTURER", label: "Manufacturer" }, { val: "MARSHALLING", label: "Marshalling" }, { val: "NVOCC", label: "NVOCC" }, { val: "OEM", label: "OEM" }, { val: "OTHER", label: "Other" }, { val: "CAPTIVE", label: "Private Party" }, { val: "RAIL", label: "Rail" }, { val: "RENTAL", label: "Rental" }, { val: "REPO AGENT", label: "Repo Agent" }, { val: "REPO YARD", label: "Repo Yard" }, { val: "INDIVIDUAL", label: "Residence" }, { val: "AUTO_PROCESSING", label: "Seasonal Transport" }, { val: "STORAGE YARD", label: "Storage Yard" }, { val: "TOW YARD", label: "Tow Yard" }, { val: "RORO", label: "Transport" }, { val: "VDC", label: "Vehicle Distribution Center" }, { val: "VPC", label: "Vehicle Processing Center" }]
const cargoList = ["Drive Away", "Heavy Equipment", "Hook On", "Luxury", "OEM", "Repo", "Salvage", "Standard Auto"]
const autoSendOptions = [{val: "[NONE]", label: "None"}, {val: "E", label: "Email"}, {val: "F", label: "Fax"}]
const statusToggles = { autosendNotify: 'Notify Email', hasDomains: 'Domains', carrierDispatchEmail: 'Dispatch Email', hasUDA: 'UDA Account', hasUDAConfig: 'UDA Config', hasUDAInvite: 'UDA Invite', is3plWireless: '3PL User', isCarrier: 'Carrier', chargeAvail: 'Charge Avail', isConsignor: 'Consignor' }

// --- STATE ---
const filters = reactive({
  accountingIds: [], cargoTypes: [], clientCreatedBy: [], attachedDocs: [], businessTypes: [], autoSendBol: [],
  businessDomains: [], containsOnlyCargo: [], carrierCargoText: '', dispatchExclude: '',
  autosendNotify: null, hasDomains: null, carrierDispatchEmail: null, hasUDA: null, hasUDAConfig: null, hasUDAInvite: null,
  is3plWireless: null, isCarrier: null, chargeAvail: null, isConsignor: null
})
const search = reactive({ docs: '', biz: '' })
const showManager = ref(false)

// --- SEARCH LOGIC ---
const isSectionVisible = (sectionTitle) => {
  if (!globalSearch.value) return true;
  const term = globalSearch.value.toLowerCase();
  
  // Define content per section for searching
  const sectionContentMap = {
    'Identification & Accounting': 'accounting ids cargo types created by identification',
    'Account Status & Flags': Object.values(statusToggles).join(' '),
    'Documents & Business Types': 'attached docs business types documents',
    'Carrier & Domains': 'auto-send bol carrier cargo types only cargo types central dispatch exclusions business domains'
  }
  
  return sectionTitle.toLowerCase().includes(term) || 
         (sectionContentMap[sectionTitle] || '').toLowerCase().includes(term);
}

const shouldOpen = (sectionTitle) => {
  if (!globalSearch.value) return sectionTitle === 'Identification & Accounting'; // Default open
  return isSectionVisible(sectionTitle);
}

// --- COMPUTED ---
const filteredDocs = computed(() => docOptions.filter(o => o.label.toLowerCase().includes(search.docs.toLowerCase())))
const filteredBiz = computed(() => bizOptions.filter(o => o.label.toLowerCase().includes(search.biz.toLowerCase())))
const activeFilterCount = computed(() => {
  return Object.values(filters).filter(v => (Array.isArray(v) && v.length > 0) || (typeof v === 'string' && v !== '') || (typeof v === 'boolean')).length
})

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
  Object.keys(filters).forEach(k => {
    if (Array.isArray(filters[k])) filters[k] = []
    else if (typeof filters[k] === 'string') filters[k] = ''
    else filters[k] = null
  })
}
const onSearch = () => console.log("Final Filter Payload:", JSON.parse(JSON.stringify(filters)))
</script>

<style scoped>
/* AMBER/ORANGE THEME STYLES */
.mini-filter-system { font-family: 'Inter', sans-serif; }
.first, .legacy-wrapper, .third { width: 100%; max-width: 380px; }
.layout { width: 100%; margin-top: 20px; display: flex; justify-content: space-between; flex-wrap: wrap; gap: 20px; }
.mini-header { padding: 12px 16px; background: #faa643; color: white; display: flex; justify-content: space-between; align-items: center; }
.list-row input { width: unset; }
.brand { display: flex; align-items: center; gap: 8px; }
.dot-indicator { width: 8px; height: 8px; background: #4ade80; border-radius: 50%; }
.title { font-weight: 700; font-size: 0.9rem; }
.count-badge { font-size: 0.7rem; background: rgba(0,0,0,0.2); padding: 2px 6px; border-radius: 4px; }
.header-actions { display: flex; gap: 8px; }
.text-btn { background: none; border: none; color: white; opacity: 0.8; font-size: 0.75rem; cursor: pointer; }
.apply-btn { background: white; color: #faa643; border: none; padding: 4px 10px; border-radius: 4px; font-weight: 700; font-size: 0.75rem; cursor: pointer; }
.accordion-container { max-height: 80vh; overflow-y: auto; }
.filter-section { border-bottom: 1px solid #f1f5f9; }
.filter-section summary { padding: 12px 16px; font-size: 0.75rem; font-weight: 700; color: #64748b; cursor: pointer; background: #f8fafc; list-style: none; }
.filter-section summary:hover { background: #f1f5f9; }
.section-content { padding: 16px 18px; background: #f8f9fc; border: 1px solid #e2e8f0; border-radius: 8px; margin-bottom: 12px; box-shadow: 0 2px 8px 0 rgba(60,72,100,0.04); }
.grid-2-col { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.field-row { margin-bottom: 12px; }
.field-row label { display: block; font-size: 0.7rem; font-weight: 600; color: #94a3b8; margin-bottom: 4px; }
.tag-wrap { border: 1px solid #e2e8f0; border-radius: 6px; padding: 4px; display: flex; flex-wrap: wrap; gap: 4px; }
.mini-tag { background: #fff8ef; color: #faa643; font-size: 0.65rem; padding: 1px 6px; border-radius: 3px; display: flex; align-items: center; border: 1px solid #ffedd5; }
.mini-tag i { margin-left: 4px; cursor: pointer; font-style: normal; font-weight: bold; }
.tag-wrap input { border: none; outline: none; font-size: 0.7rem; flex: 1; min-width: 40px; }
.compact-multi { width: 100%; height: 80px; font-size: 0.75rem; border: 1px solid #e2e8f0; border-radius: 6px; }
.compact-multi-small { width: 100%; height: 50px; font-size: 0.75rem; border: 1px solid #e2e8f0; border-radius: 6px; }
.mini-toggle-box { border: 1px solid #f1f5f9; padding: 6px; border-radius: 6px; }
.tiny-label { font-size: 0.65rem; display: block; margin-bottom: 4px; color: #475569; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.micro-tri-state { display: flex; background: #f1f5f9; border-radius: 4px; padding: 2px; }
.micro-tri-state button { flex: 1; border: none; background: transparent; font-size: 0.6rem; padding: 2px 0; cursor: pointer; border-radius: 3px; color: #94a3b8; }
.micro-tri-state button.active { background: white; color: #faa643; font-weight: 700; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.list-container { border: 1px solid #e2e8f0; border-radius: 6px; overflow: hidden; }
.inner-search { width: 100%; border: none; border-bottom: 1px solid #e2e8f0; padding: 6px; font-size: 0.7rem; background: #f8fafc; }
.micro-scroll { height: 100px; overflow-y: auto; }
.list-row { padding: 4px 8px; font-size: 0.7rem; display: flex; align-items: center; gap: 6px; cursor: pointer; }
.list-row:hover { background: #fff8ef; }
.mini-area { width: 100%; border: 1px solid #e2e8f0; border-radius: 6px; font-size: 0.75rem; padding: 6px; }
.inline-checks { display: flex; gap: 10px; font-size: 0.7rem; }
.legacy-wrapper { padding: 10px; }

/* SEARCH BAR STYLES */
.main-search-container {
  padding: 10px 16px;
  background: #fff;
  border-bottom: 1px solid #e2e8f0;
}
.global-search-input {
  width: 100%;
  padding: 8px 12px;
  border: 2px solid #f1f5f9;
  max-width: 200px;
  border-radius: 8px;
  font-size: 0.8rem;
  outline: none;
  transition: border-color 0.2s;
}
.global-search-input:focus {
  border-color: #faa643;
}

/* Accordion arrow styles */
.accordion-arrow {
  display: inline-block;
  width: 10px;
  height: 10px;
  margin-right: 8px;
  border-right: 2px solid #faa643;
  border-bottom: 2px solid #faa643;
  transform: rotate(45deg);
  transition: transform 0.2s;
  position: relative;
  left: 95%;
}
details[open] > summary .accordion-arrow {
  transform: rotate(-135deg);
}

@media (prefers-color-scheme: dark) {
  .mini-filter-system {
    background: #181a23;
    color: #e0e6f7;
  }
  .mini-header {
    background: #23204a;
    color: #e0e6f7;
  }
  .brand .dot-indicator {
    background: #4ade80;
  }
  .count-badge {
    background: rgba(255,255,255,0.12);
    color: #c7d2fe;
  }
  .header-actions .text-btn {
    color: #e0e6f7;
  }
  .header-actions .apply-btn {
    background: #faa643;
    color: #23204a;
  }
  .main-search-container {
    background: #23263a;
    border-bottom: 1px solid #23204a;
  }
  .global-search-input {
    background: #181a23;
    color: #e0e6f7;
    border-color: #23204a;
  }
  .global-search-input:focus {
    border-color: #faa643;
  }
  .accordion-container {
    background: #181a23;
  }
  .filter-section {
    border-bottom: 1px solid #23204a;
  }
  .filter-section summary {
    background: #23263a;
    color: #a3aed6;
  }
  .filter-section summary:hover {
    background: #23204a;
  }
  .section-content {
    background: #23263a;
    border: 1px solid #23204a;
    color: #e0e6f7;
    box-shadow: 0 2px 8px 0 rgba(30,40,60,0.18);
  }
  .mini-tag {
    background: #23204a;
    color: #faa643;
    border-color: #353a50;
  }
  .mini-tag i {
    color: #a3aed6;
  }
  .tag-wrap {
    border: 1px solid #23204a;
    background: #181a23;
  }
  .tag-wrap input {
    background: #181a23;
    color: #e0e6f7;
  }
  .compact-multi, .compact-multi-small {
    background: #181a23;
    color: #e0e6f7;
    border: 1px solid #23204a;
  }
  .mini-toggle-box {
    border: 1px solid #23204a;
    background: #23263a;
  }
  .tiny-label {
    color: #a3aed6;
  }
  .micro-tri-state {
    background: #23204a;
  }
  .micro-tri-state button {
    color: #a3aed6;
  }
  .micro-tri-state button.active {
    background: #faa643;
    color: #23204a;
  }
  .list-container {
    border: 1px solid #23204a;
    background: #181a23;
  }
  .inner-search {
    background: #23263a;
    color: #e0e6f7;
    border-bottom: 1px solid #23204a;
  }
  .micro-scroll {
    background: #181a23;
  }
  .list-row {
    color: #e0e6f7;
  }
  .list-row:hover {
    background: #23204a;
  }
  .mini-area {
    background: #181a23;
    color: #e0e6f7;
    border: 1px solid #23204a;
  }
  .inline-checks label {
    color: #a3aed6;
  }
  .legacy-wrapper {
    background: #181a23;
  }
  .accordion-arrow {
    border-right: 2px solid #faa643;
    border-bottom: 2px solid #faa643;
  }
}
</style>
<style scoped>
.quick-filters-heading {
  font-size: 1.4rem;
  font-weight: 700;
  color: #147393;
  margin-bottom: 0.5rem;
  margin-top: 0.5rem;
  letter-spacing: 0.01em;
  font-family: 'Inter', -apple-system, sans-serif;
}
</style>

