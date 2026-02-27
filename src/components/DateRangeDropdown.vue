<template>
  <div class="date-range-heading">Date Range</div>
  <div class="filter-container">
    <div class="filter-bar">
      <div class="filter-pill" :class="{ 'is-active': dateType !== '-' }">
        <span class="pill-label">Type</span>
        <select v-model="dateType" class="pill-select">
          <option value="-">No Filter</option>
          <option value="date_filter_ActivationDate_ID">Activation Date</option>
          <option value="CallbackDate">Callback Date</option>
          <option value="CreatedDate">Client Creation Date</option>
          <option value="ClientEventDate">Contact Status Date</option>
          <option value="InsExp">Insurance Expiration</option>
          <option value="ProfileChanged">Profile Changed</option>
        </select>
      </div>

      <div class="filter-pill" :class="{ 'is-active': dateRange !== '' && dateRange !== '[NONE]' }">
        <span class="pill-label">Range</span>
        <select v-model="dateRange" class="pill-select">
          <option value="" disabled hidden>Select range</option>
          <optgroup label="Relative">
            <option value="Today">Today</option>
            <option value="Yesterday">Yesterday</option>
            <option value="Tomorrow">Tomorrow</option>
            <option value="Yesterday and Today">Yesterday and Today</option>
          </optgroup>
          <optgroup label="Next">
            <option value="Next 3 Days">Next 3 Days</option>
            <option value="Next 4 Days">Next 4 Days</option>
            <option value="Next 5 Days">Next 5 Days</option>
            <option value="Next 7 Days">Next 7 Days</option>
            <option value="Next 14 Days">Next 14 Days</option>
            <option value="Next 30 Days">Next 30 Days</option>
            <option value="Next 120 Days">Next 120 Days</option>
          </optgroup>
          <optgroup label="Standard Periods">
            <option value="This Week">This Week</option>
            <option value="Last Week">Last Week</option>
            <option value="Next Week">Next Week</option>
            <option value="This Week and Last Week">This Week and Last Week</option>
            <option value="This Month">This Month</option>
            <option value="Next Month">Next Month</option>
            <option value="Last Month">Last Month</option>
            <option value="This Quarter">This Quarter</option>
            <option value="This Year">This Year</option>
          </optgroup>
          <option value="Before">Before</option>
          <option value="After">After</option>
          <option value="[NONE]">[NONE]</option>
        </select>
      </div>

      <div class="manual-group">
        <div class="filter-pill calendar-pill" :class="{ 'is-active': fromDate }">
          <span class="pill-label">From</span>
          <input type="text" :value="fromDate" @focus="activeCalendar = 'from'" placeholder="YYYY-MM-DD" readonly />
          
          <div v-if="activeCalendar === 'from'" class="calendar-popup">
             <div class="calendar-header">
                <button @click.stop="changeMonth(-1)">&lt;</button>
                <span>{{ monthLabel }} {{ viewYear }}</span>
                <button @click.stop="changeMonth(1)">&gt;</button>
             </div>
             <table class="calendar-table">
                <thead><tr><th v-for="d in daysOfWeek" :key="d">{{d}}</th></tr></thead>
                <tbody>
                  <tr v-for="(week, i) in calendarGrid" :key="i">
                    <td v-for="(day, j) in week" :key="j" 
                        :class="{ 'selected': fromDate === formatDate(day), 'empty': !day }"
                        @click.stop="selectDay(day, 'from')">
                      {{ day }}
                    </td>
                  </tr>
                </tbody>
             </table>
          </div>
        </div>

        <div class="filter-pill calendar-pill" :class="{ 'is-active': toDate }">
          <span class="pill-label">To</span>
          <input type="text" :value="toDate" @focus="activeCalendar = 'to'" placeholder="YYYY-MM-DD" readonly />
          
          <div v-if="activeCalendar === 'to'" class="calendar-popup">
             <div class="calendar-header">
                <button @click.stop="changeMonth(-1)">&lt;</button>
                <span>{{ monthLabel }} {{ viewYear }}</span>
                <button @click.stop="changeMonth(1)">&gt;</button>
             </div>
             <table class="calendar-table">
                <thead><tr><th v-for="d in daysOfWeek" :key="d">{{d}}</th></tr></thead>
                <tbody>
                  <tr v-for="(week, i) in calendarGrid" :key="i">
                    <td v-for="(day, j) in week" :key="j" 
                        :class="{ 'selected': toDate === formatDate(day), 'empty': !day }"
                        @click.stop="selectDay(day, 'to')">
                      {{ day }}
                    </td>
                  </tr>
                </tbody>
             </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

// State
const dateType = ref('-');
const dateRange = ref('');
const fromDate = ref('');
const toDate = ref('');
const activeCalendar = ref(null); // 'from' or 'to' or null

const viewMonth = ref(new Date().getMonth());
const viewYear = ref(new Date().getFullYear());

const daysOfWeek = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
const monthLabel = computed(() => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'][viewMonth.value]);

// Calendar Logic
const calendarGrid = computed(() => {
  const firstDay = new Date(viewYear.value, viewMonth.value, 1).getDay();
  const daysInMonth = new Date(viewYear.value, viewMonth.value + 1, 0).getDate();
  const grid = [];
  let day = 1;
  for (let i = 0; i < 6; i++) {
    const week = [];
    for (let j = 0; j < 7; j++) {
      if (i === 0 && j < firstDay) week.push(null);
      else if (day > daysInMonth) week.push(null);
      else week.push(day++);
    }
    grid.push(week);
  }
  return grid;
});

const changeMonth = (step) => {
  viewMonth.value += step;
  if (viewMonth.value > 11) { viewMonth.value = 0; viewYear.value++; }
  if (viewMonth.value < 0) { viewMonth.value = 11; viewYear.value--; }
};

const formatDate = (day) => {
  if (!day) return null;
  return `${viewYear.value}-${String(viewMonth.value + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
};

const selectDay = (day, target) => {
  if (!day) return;
  const formatted = formatDate(day);
  if (target === 'from') fromDate.value = formatted;
  else toDate.value = formatted;
  activeCalendar.value = null;
};

// Close calendar on outside click
const closePopups = (e) => {
  if (!e.target.closest('.calendar-pill')) activeCalendar.value = null;
};
onMounted(() => document.addEventListener('mousedown', closePopups));
onBeforeUnmount(() => document.removeEventListener('mousedown', closePopups));
</script>

<style scoped>
.filter-container {
  padding: 20px;
  background: #f8f9fc;
  margin-top: 20px;
}

.filter-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
}

.filter-pill {
  position: relative;
  display: flex;
  align-items: center;
  background: white;
  border: 1.5px solid #e2e8f0;
  border-radius: 8px;
  padding: 4px 12px;
  transition: 0.2s;
}

.filter-pill:hover { border-color: #faa643; }
.filter-pill.is-active {
  border-color: #faa643;
  background: #fff8ef;
}

.pill-label {
  font-size: 11px;
  font-weight: 800;
  color: #94a3b8;
  text-transform: uppercase;
  margin-right: 8px;
  border-right: 1px solid #e2e8f0;
  padding-right: 8px;
}

    .is-active .pill-label { color: #faa643; border-right-color: #faa643; }

.pill-select, .calendar-pill input {
  border: none;
  background: transparent;
  font-size: 13px;
  color: #1e293b;
  font-weight: 600;
  outline: none;
  cursor: pointer;
}

.manual-group {
  display: flex;
  gap: 8px;
  border-left: 2px solid #e2e8f0;
  padding-left: 10px;
}

/* Calendar Styling */
.calendar-popup {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  z-index: 100;
  background: #fff8ef;
  border: 1px solid #faa643;
  border-radius: 12px;
  padding: 12px;
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-weight: bold;
  color: #faa643;
}

.calendar-table td {
  padding: 6px;
  text-align: center;
  cursor: pointer;
  border-radius: 4px;
  font-size: 12px;
}

.calendar-table td:hover:not(.empty) { background: #fff8ef; color: #faa643; }
.calendar-table td.selected { background: #faa643; color: white; }

@media (max-width: 768px) {
  .manual-group { border-left: none; padding-left: 0; width: 100%; }
  .filter-pill { flex: 1; min-width: 140px; }
}
/* Dark mode support */
@media (prefers-color-scheme: dark) {
  .filter-container {
    background: #181a20;
  }
  .filter-bar, .manual-group {
    border-left-color: #23263a;
  }
  .filter-pill {
    background: #23263a;
    border-color: #353a50;
    color: #e0e6f5;
  }
  .filter-pill.is-active {
    background: #23204a;
    border-color: #a78bfa;
  }
  .pill-label {
    color: #a3aed6;
    border-right-color: #353a50;
  }
  .is-active .pill-label {
    color: #faa643;
    border-right-color: #a78bfa;
  }
  .pill-select, .calendar-pill input {
    background: transparent;
    color: #e0e6f5;
  }
  .pill-select option,
  .pill-select optgroup {
    background: #23263a;
    color: #e0e6f5;
  }
  .calendar-popup {
    background: #23263a;
    border-color: #a78bfa;
    color: #e0e6f5;
    box-shadow: 0 10px 25px -5px rgba(0,0,0,0.7);
  }
  .calendar-header {
    color: #a78bfa;
  }
  .calendar-table td {
    color: #e0e6f5;
  }
  .calendar-table td:hover:not(.empty) {
    background: #2d2e4a;
    color: #a78bfa;
  }
  .calendar-table td.selected {
    background: #faa643;
    color: #23263a;
  }
}
</style>
<style scoped>
.date-range-heading {
  font-size: 1.4rem;
  font-weight: 700;
  color: #147393;
  margin-bottom: 0.5rem;
  margin-top: 0.5rem;
  letter-spacing: 0.01em;
  font-family: 'Inter', -apple-system, sans-serif;
}
</style>
