<template>
  <div class="date-manual-select">
    <input type="text" v-model="dateString" @focus="showCalendar = true" placeholder="Select date" readonly />
    <div v-if="showCalendar" class="calendar-popup">
      <div class="calendar-header">
        <button @click="prevMonth">&lt;&lt;</button>
        <span>{{ monthLabel }} {{ year }}</span>
        <button @click="nextMonth">&gt;&gt;</button>
      </div>
      <table class="calendar-table">
        <thead>
          <tr>
            <th v-for="d in daysOfWeek" :key="d">{{ d }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="week in calendar" :key="week">
            <td v-for="dayObj in week" :key="dayObj.day"
                @click="selectDay(dayObj.day, $event)"
                @keydown.enter="selectDay(dayObj.day, $event)"
                @keydown.space="selectDay(dayObj.day, $event)"
                tabindex="0"
                :class="{ selected: dayObj.day === selectedDay }">
              {{ dayObj.day || '' }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';
const showCalendar = ref(false);
const selectedDay = ref('');
const selectedMonth = ref(new Date().getMonth() + 1);
const selectedYear = ref(new Date().getFullYear());
const dateString = ref('');
const daysOfWeek = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
const monthLabel = computed(() => {
  const labels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
  return labels[selectedMonth.value - 1];
});
const year = computed(() => selectedYear.value);

function prevMonth() {
  if (selectedMonth.value === 1) {
    selectedMonth.value = 12;
    selectedYear.value--;
  } else {
    selectedMonth.value--;
  }
}
function nextMonth() {
  if (selectedMonth.value === 12) {
    selectedMonth.value = 1;
    selectedYear.value++;
  } else {
    selectedMonth.value++;
  }
}
function selectDay(day, event) {
  if (!day) return;
  selectedDay.value = day;
  dateString.value = `${year.value}-${String(selectedMonth.value).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
  showCalendar.value = false;
  if (event) {
    event.stopPropagation();
    event.preventDefault();
  }
}

function handleClickOutside(event) {
  const calendar = document.querySelector('.calendar-popup');
  const input = document.querySelector('.date-manual-select input');
  if (showCalendar.value && calendar && !calendar.contains(event.target) && input && !input.contains(event.target)) {
    showCalendar.value = false;
  }
}

onMounted(() => {
  document.addEventListener('mousedown', handleClickOutside);
});
onBeforeUnmount(() => {
  document.removeEventListener('mousedown', handleClickOutside);
});

const calendar = computed(() => {
  const weeks = [];
  const firstDay = new Date(year.value, selectedMonth.value - 1, 1).getDay();
  const daysInMonth = new Date(year.value, selectedMonth.value, 0).getDate();
  let dayNum = 1 - firstDay;
  for (let w = 0; w < 6; w++) {
    const week = [];
    for (let d = 0; d < 7; d++) {
      week.push({ day: dayNum > 0 && dayNum <= daysInMonth ? dayNum : null });
      dayNum++;
    }
    weeks.push(week);
  }
  return weeks;
});
</script>

<style scoped>
.date-manual-select {
  margin-top: 10px;
  padding: 12px;
  background: rgba(255,255,255,0.95); /* Increased opacity for better visibility */
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.12); /* Slightly stronger shadow */
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  position: relative;
}
.date-manual-select input {
  width: 200px;
  padding: 10px;
  border-radius: 6px;
  border: 1.5px solid #7c4dff; /* More visible border */
  font-size: 16px;
  background: #fff;
  color: #222;
  box-shadow: 0 1px 4px rgba(124,77,255,0.08);
}
.calendar-popup {
  margin-top: 8px;
  background: #fff;
  border: 2px solid #7c4dff; /* More visible border */
  border-radius: 10px;
  padding: 12px;
  position: absolute;
  z-index: 2550000;
  box-shadow: 0 4px 16px rgba(124,77,255,0.12);
}
.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  font-weight: bold;
  color: #7c4dff;
}
.calendar-table {
  width: 100%;
  border-collapse: collapse;
  background: #faf8ff;
}
.calendar-table th, .calendar-table td {
  width: 36px;
  height: 36px;
  text-align: center;
  border: 1px solid #e0d7fa;
  cursor: pointer;
  font-size: 15px;
  color: #333;
  background: #fff;
  transition: background 0.2s, color 0.2s;
}
.calendar-table th {
  background: #ede7f6;
  color: #7c4dff;
}
.calendar-table td.selected {
  background: #7c4dff;
  color: #fff;
  border-radius: 50%;
}
.calendar-table td:hover {
  background: #ede7f6;
  color: #7c4dff;
}
</style>
