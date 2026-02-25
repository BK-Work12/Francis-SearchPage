<template>
  <div class="date-input-calendar">
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
            <td v-for="dayObj in week" :key="dayObj.day" @click="selectDay(dayObj.day)" :class="{ selected: dayObj.day === selectedDay }">
              {{ dayObj.day || '' }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
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
function selectDay(day) {
  if (!day) return;
  selectedDay.value = day;
  dateString.value = `${year.value}-${String(selectedMonth.value).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
  showCalendar.value = false;
}
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
.date-input-calendar {
  margin-top: 10px;
  padding: 12px;
  background: rgba(255,255,255,0.7);
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  position: relative;
}
.date-input-calendar input {
  width: 180px;
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #ccc;
  font-size: 16px;
}
.calendar-popup {
  margin-top: 8px;
  background: #fff;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 8px;
  position: absolute;
  z-index: 2550000;
}
.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 4px;
}
.calendar-table {
  width: 100%;
  border-collapse: collapse;
}
.calendar-table th, .calendar-table td {
  width: 32px;
  height: 32px;
  text-align: center;
  border: 1px solid #eee;
  cursor: pointer;
}
.calendar-table td.selected {
  background: #7c4dff;
  color: #fff;
  border-radius: 50%;
}
</style>
