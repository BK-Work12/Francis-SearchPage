<template>
  <div class="mini-sidebar">
    <header class="sidebar-header">
      <div class="status-indicator">
        <span class="dot"></span>
        <span class="count">{{ getTabCount('location') + getTabCount('uda') }} Active Filters</span>
      </div>
      <button class="reset-link" @click="resetAll">Reset All</button>
    </header>

    <div class="search-box">
      <input v-model="searchQuery" placeholder="Search parameters..." />
    </div>

    <div class="accordion-body">
      <details open class="filter-group">
        <summary>
          <span class="accordion-arrow"></span>
          <span class="label">Location</span>
          <span v-if="getTabCount('location')" class="badge">{{ getTabCount('location') }}</span>
        </summary>
        
        <div class="content">
          <div class="mini-input-group">
            <label>Area Codes</label>
            <div class="chip-wall">
              <span v-for="(code, i) in filterState.areaCodes" :key="i" class="mini-chip">
                {{ code }} <i @click="removeListSubItem('areaCodes', i)">×</i>
              </span>
              <input @keydown.enter="addListSubItem($event, 'areaCodes')" placeholder="Add code..." />
            </div>
          </div>

          <div class="mini-input-group">
            <label>Cities</label>
            <div class="chip-wall">
              <span v-for="(city, i) in filterState.cities" :key="i" class="mini-chip">
                {{ city }} <i @click="removeListSubItem('cities', i)">×</i>
              </span>
              <input @keydown.enter="addListSubItem($event, 'cities')" placeholder="Add city..." />
            </div>
          </div>

          <div class="mini-input-group">
            <label>Countries ({{ filterState.countries.length }})</label>
            <div class="mini-scroll-list">
              <div 
                v-for="country in filteredCountries" :key="country" 
                class="list-row" :class="{ 'selected': filterState.countries.includes(country) }"
                @click="toggleCountry(country)"
              >
                <input type="checkbox" :checked="filterState.countries.includes(country)" />
                <span>{{ country }}</span>
              </div>
            </div>
          </div>
        </div>
      </details>

      <details class="filter-group">
        <summary>
          <span class="accordion-arrow"></span>
          <span class="label">UDA Settings</span>
          <span v-if="getTabCount('uda')" class="badge">{{ getTabCount('uda') }}</span>
        </summary>
        
        <div class="content">
          <div v-for="uda in filteredUDAs" :key="uda.key" class="mini-uda-row">
            <span class="uda-text">{{ cleanLabel(uda.label) }}</span>
            <div class="micro-tri-state">
              <button :class="{ active: filterState.udas[uda.key] === null }" @click="filterState.udas[uda.key] = null">Any</button>
              <button :class="{ active: filterState.udas[uda.key] === false }" @click="filterState.udas[uda.key] = false">No</button>
              <button :class="{ active: filterState.udas[uda.key] === true }" @click="filterState.udas[uda.key] = true">Yes</button>
            </div>
          </div>
        </div>
      </details>
    </div>

    <footer class="sidebar-footer">
      <button class="apply-btn" @click="applyFilters">Apply Configuration</button>
    </footer>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'

const searchQuery = ref('')

// DATA PRESERVATION: All 100+ Options kept intact
const ALL_COUNTRIES = ["[NONE]", "AFGHANISTAN", "ALACHUA", "ALAMEDA", "ALBANIA", "ALGERIA", "AMERICAN SAMOA", "ANDORRA", "ANGOLA", "ANGUILLA", "ANTARCTICA", "ANTIGUA ", "ARGENTINA", "ARMENIA", "ARUBA", "AUSTRALIA", "AUSTRIA", "AZERBAIJAN", "BAHAMAS", "BAHRAIN", "BANGLADESH", "BARBADOS", "BARBUDA", "BELARUS", "BELGIUM", "BELIZE", "BENIN", "BERMUDA", "BHUTAN", "BOLIVIA", "BOSNIA ", "BOTSWANA", "BOUVET ISLAND", "BRAZIL", "BRUNEI DARUSSALAM", "BULGARIA", "BURKINA FASO", "BURUNDI", "CAICOS ISLANDS", "CAMBODIA", "CAMEROON", "CANADA", "CAPE VERDE", "CAYMAN ISLANDS", "CENTRAL AFRICAN REP.", "CHAD", "CHILE", "CHINA", "CHRISTMAS ISLAND", "COCOS ISLANDS", "COLOMBIA", "COMOROS", "CONGO", "COOK ISLANDS", "COSTA RICA", "COTE D'IVOIRE", "CROATIA", "CUBA", "CYPRUS", "CZECH REPUBLIC", "DENMARK", "DJIBOUTI", "DOMINICA", "DOMINICAN REPUBLIC", "EAST TIMOR", "ECUADOR", "EGYPT", "EL SALVADOR", "EQUATORIAL GUINEA", "ERITREA", "ESTONIA", "ETHIOPIA", "FALKLAND ISLANDS", "FAROE ISLANDS", "FIJI", "FINLAND", "FRANCE", "FRANCE, METROPOLITAN", "FRENCH GUIANA", "FRENCH POLYNESIA", "FUTUNA ISLANDS", "GABON", "GAMBIA", "GEORGIA", "GERMANY", "GHANA", "GIBRALTAR", "GREECE", "GREENLAND", "GRENADA", "GUADELOUPE", "GUAM", "GUATEMALA", "GUINEA", "GUINEA-BISSAU", "GUYANA", "HAITI", "HEARD ISLANDS", "HERNANDO", "HERZEGOWINA", "HONDURAS", "HONG KONG", "HUNGARY", "ICELAND", "INDIA", "INDONESIA", "IRAN", "IRAQ", "IRELAND", "ISRAEL", "ITALY", "JAMAICA", "JAN MAYEN ISLANDS", "JAPAN", "JORDAN", "KAZAKHSTAN", "KENYA", "KIRIBATI", "KOREA", "KOREA, REPUBLIC OF", "KUWAIT", "KYRGYZSTAN", "LATVIA", "LEBANON", "LESOTHO", "LIBERIA", "LIBYN RB JMHIRIY", "LIECHTENSTEIN", "LITHUANIA", "LUXEMBOURG", "LUZERNE", "MACAU", "MACEDONIA", "MADAGASCAR", "MALAWI", "MALAYSIA", "MALDIVES", "MALI", "MALTA", "MARSHALL ISLANDS", "MARTINIQUE", "MAURITANIA", "MAURITIUS", "MAYOTTE", "MC DONALD ISLANDS", "MEXICO", "MICRONESIA", "MOLDOVA, REPUBLIC OF", "MONACO", "MONGOLIA", "MONTSERRAT", "MOROCCO", "MOZAMBIQUE", "MYANMAR", "N. MARIANA ISLANDS", "NAMIBIA", "NAURU", "NEPAL", "NETHERLANDS", "NETHERLANDS ANTILLES", "NEVIS", "NEW CALEDONIA", "NEW ZEALAND", "NICARAGUA", "NIGER", "NIGERIA", "NIUE", "NORFOLK ISLAND", "NORWAY", "OMAN", "PAKISTAN", "PALAU", "PANAMA", "PAPUA NEW GUINEA", "PARAGUAY", "PERU", "PHILIPPINES", "PITCAIRN", "POLAND", "PORTUGAL", "PRINCIPE", "PUERTO RICO", "QATAR", "REUNION", "ROMANIA", "RUSSIAN FEDERATION", "RWANDA", "SAINT KITTS ", "SAINT LUCIA", "SAINT VINCENT", "SAMOA", "SAN MARINO", "SAO TOME ", "SAUDI ARABIA", "SENEGAL", "SERBIA", "SEYCHELLES", "SIERRA LEONE", "SINGAPORE", "SLOVAKIA", "SLOVENIA", "SOLOMON ISLANDS", "SOMALIA", "SOUTH AFRICA", "SPAIN", "SRI LANKA", "ST. HELENA", "ST. MIQUELON", "ST. PIERRE ", "SUDAN", "SURINAME", "SVALBARD ISLANDS", "SWAZILAND", "SWEDEN", "SWITZERLAND", "SYRIAN ARAB REPUBLIC", "TAIWAN", "TAJIKISTAN", "TANZANIA", "THAILAND", "TOBAGO", "TOGO", "TOKELAU", "TONGA", "TRINIDAD ", "TUNISIA", "TURKEY", "TURKMENISTAN", "TURKS ISLANDS", "TUVALU", "UGANDA", "UKRAINE", "UNITED ARAB EMIRATES", "UNITED KINGDOM", "URUGUAY", "USA", "UZBEKISTAN", "VANUATU", "VATICAN", "VENEZUELA", "VIET NAM", "VIRGIN ISLANDS (UK)", "VIRGIN ISLANDS (US)", "WALLIS ISLANDS", "WESTERN SAHARA", "YEMEN", "YUGOSLAVIA", "ZAIRE", "ZAMBIA", "ZIMBABWE"]

const UDA_LIST = [
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

const filterState = reactive({
  areaCodes: [],
  cities: [],
  countries: [],
  udas: UDA_LIST.reduce((acc, curr) => { acc[curr.key] = null; return acc }, {})
})

// --- LOGIC ---
const cleanLabel = (str) => str.replace(/^UDA-?/, '').replace(/-/g, ' ')
const filteredUDAs = computed(() => UDA_LIST.filter(u => u.label.toLowerCase().includes(searchQuery.value.toLowerCase())))
const filteredCountries = computed(() => ALL_COUNTRIES.filter(c => c.toLowerCase().includes(searchQuery.value.toLowerCase())))

const getTabCount = (tabId) => {
  if (tabId === 'location') return filterState.areaCodes.length + filterState.cities.length + filterState.countries.length
  return Object.values(filterState.udas).filter(v => v !== null).length
}

const toggleCountry = (country) => {
  const index = filterState.countries.indexOf(country)
  index > -1 ? filterState.countries.splice(index, 1) : filterState.countries.push(country)
}

const addListSubItem = (e, key) => {
  const val = e.target.value.trim()
  if (val && !filterState[key].includes(val)) { filterState[key].push(val); e.target.value = '' }
}

const removeListSubItem = (key, index) => filterState[key].splice(index, 1)

const resetAll = () => {
  filterState.areaCodes = []; filterState.cities = []; filterState.countries = []
  Object.keys(filterState.udas).forEach(k => filterState.udas[k] = null)
  searchQuery.value = ''
}

const applyFilters = () => console.log('Payload:', JSON.parse(JSON.stringify(filterState)))
</script>

<style scoped>
.mini-sidebar {
  background: #fff; border: 1px solid #e2e8f0; border-radius: 8px;
  display: flex; flex-direction: column; height: 100%; font-family: 'Inter', sans-serif;
}
.list-row input {
  width: unset;
}
.sidebar-header {
  padding: 12px 16px; display: flex; justify-content: space-between; align-items: center;
  border-bottom: 1px solid #f1f5f9;
}
.status-indicator { display: flex; align-items: center; gap: 6px; font-size: 0.75rem; color: #64748b; font-weight: 600; }
.dot { width: 6px; height: 6px; background: #10b981; border-radius: 50%; }
.reset-link { background: none; border: none; color: #7c4dff; cursor: pointer; font-size: 0.7rem; font-weight: 700; }

.search-box { padding: 8px 12px; }
.search-box input {
  width: 100%; max-width: 200px; padding: 6px 10px; border: 1px solid #e2e8f0; border-radius: 4px; font-size: 0.8rem;
}

.accordion-body { flex: 1; overflow-y: auto; }

.filter-group { border-bottom: 1px solid #f1f5f9; }
.filter-group summary {
  padding: 10px 16px; cursor: pointer; display: flex; align-items: center;
  background: #f8fafc; list-style: none; outline: none;
}
.filter-group summary::-webkit-details-marker { display: none; }
.filter-group .label { font-size: 0.75rem; font-weight: 700; color: #475569; text-transform: uppercase; }
.badge { background: #7c4dff; color: #fff; font-size: 0.65rem; padding: 1px 6px; border-radius: 10px; }

.content {
   padding: 12px 16px; 
  max-height: 200px; overflow-y: auto;
  }

.mini-input-group { margin-bottom: 12px; }
.mini-input-group label { display: block; font-size: 0.7rem; font-weight: 600; color: #94a3b8; margin-bottom: 4px; }

.chip-wall {
  border: 1px solid #e2e8f0; border-radius: 4px; padding: 4px; display: flex; flex-wrap: wrap; gap: 4px;
}
.mini-chip {
  background: #f1f5f9; color: #475569; padding: 2px 6px; border-radius: 3px; font-size: 0.7rem; display: flex; align-items: center;
}
.mini-chip i { margin-left: 4px; cursor: pointer; font-style: normal; font-weight: bold; }
.chip-wall input { border: none; outline: none; font-size: 0.75rem; flex: 1; min-width: 60px; }

.mini-scroll-list { border: 1px solid #e2e8f0; max-height: 120px; overflow-y: auto; border-radius: 4px; }
.list-row { padding: 4px 8px; font-size: 0.75rem; display: flex; gap: 8px; cursor: pointer; }
.list-row.selected { background: #f3f0ff; color: #7c4dff; }

.mini-uda-row {
  display: flex; justify-content: space-between; align-items: center; padding: 6px 0; border-bottom: 1px solid #f8fafc;
}
.uda-text { font-size: 0.75rem; color: #334155; padding-right: 8px; line-height: 1.2; }

.micro-tri-state { display: flex; background: #f1f5f9; padding: 2px; border-radius: 4px; min-width: 110px; }
.micro-tri-state button {
  flex: 1; border: none; background: transparent; padding: 3px 0; font-size: 0.65rem; cursor: pointer; color: #94a3b8;
}
.micro-tri-state button.active { background: #fff; color: #7c4dff; font-weight: bold; border-radius: 3px; box-shadow: 0 1px 2px rgba(0,0,0,0.05); }

.sidebar-footer { padding: 12px; border-top: 1px solid #e2e8f0; }
.apply-btn {
  width: 100%; background: #7c4dff; color: #fff; border: none; padding: 8px; border-radius: 6px; font-weight: 700; font-size: 0.8rem; cursor: pointer;
}
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
