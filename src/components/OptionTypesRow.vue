<template>
  <div class="filter-row">
    
    <div class="filter-pill" :class="{ 'is-active': countActive('left') > 0 }" v-click-outside="() => close('types')">
      <span class="pill-label">Types</span>
      <div class="pill-trigger" @click="toggle('types')">
        <span class="pill-value">
          {{ formatLabel(leftCol, 'left', 'All Types') }}
          <span v-if="countActive('left') > 1" class="pill-count">+{{ countActive('left') - 1 }}</span>
        </span>
        <span class="chevron" :class="{ open: activePop === 'types' }"></span>
      </div>
      <div v-if="activePop === 'types'" class="pill-dropdown">
        <label v-for="item in leftCol" :key="item.id" class="drop-item">
          <input type="checkbox" v-model="selected[item.id]" :value="item.value" />
          {{ item.label }}
        </label>
      </div>
    </div>

    <div class="filter-pill" :class="{ 'is-active': companyType !== 'All' }" v-click-outside="() => close('company')">
      <span class="pill-label">Company</span>
      <div class="pill-trigger" @click="toggle('company')">
        <span class="pill-value">{{ getRadioLabel }}</span>
        <span class="chevron" :class="{ open: activePop === 'company' }"></span>
      </div>
      <div v-if="activePop === 'company'" class="pill-dropdown">
        <label v-for="item in middleCol" :key="item.id" class="drop-item">
          <input type="radio" v-model="companyType" name="companyType" :value="item.value" />
          {{ item.label }}
        </label>
      </div>
    </div>

    <div class="filter-pill" :class="{ 'is-active': countActive('right') > 0 }" v-click-outside="() => close('service')">
      <span class="pill-label">Service</span>
      <div class="pill-trigger" @click="toggle('service')">
        <span class="pill-value">
          {{ formatLabel(rightCol, 'right', 'None') }}
          <span v-if="countActive('right') > 1" class="pill-count">+{{ countActive('right') - 1 }}</span>
        </span>
        <span class="chevron" :class="{ open: activePop === 'service' }"></span>
      </div>
      <div v-if="activePop === 'service'" class="pill-dropdown">
        <label v-for="item in rightCol" :key="item.id" class="drop-item">
          <input type="checkbox" v-model="selected[item.id]" :value="item.value" />
          {{ item.label }}
        </label>
      </div>
    </div>

    <div class="filter-pill" :class="{ 'is-active': selected['LienHolder'] }">
      <label class="pill-trigger simple-label">
        <input type="checkbox" v-model="selected['LienHolder']" value="LienHolder" />
        <span class="pill-value">Lien Holder</span>
      </label>
    </div>

  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue';

const selected = reactive({});
const companyType = ref('All');
const activePop = ref(null);

const leftCol = [
  { id: 'Carrier', value: 'Carrier', label: 'Carrier' },
  { id: 'Consignor', value: 'Consignor', label: 'Consignor' },
  { id: 'Destination', value: 'Destination', label: 'Destination' },
  { id: 'Origin', value: 'Origin', label: 'Origin' },
  { id: 'Shipper', value: 'Shipper', label: 'Shipper' },
];

const middleCol = [
  { id: 'All', value: 'All', label: 'All' },
  { id: 'Location', value: 'Location', label: 'Location Only' },
  { id: 'POV', value: 'POV', label: 'POV' },
  { id: 'Standard', value: 'Standard', label: 'Company' },
];

const rightCol = [
  { id: 'CallbackExpired', value: 'Y', label: 'Callback Expired' },
  { id: 'HasCallback', value: 'Y', label: 'Callback Exists' },
  { id: 'ServiceProblem', value: 'Y', label: 'Service Problem' },
  { id: 'Terminal', value: 'Y', label: 'Terminal' },
];

// Helper Logic
const toggle = (val) => activePop.value = activePop.value === val ? null : val;
const close = (val) => { if(activePop.value === val) activePop.value = null; };

const countActive = (colName) => {
  const target = colName === 'left' ? leftCol : rightCol;
  return target.filter(item => selected[item.id]).length;
};

const formatLabel = (items, colName, placeholder) => {
  const active = items.find(item => selected[item.id]);
  return active ? active.label : placeholder;
};

const getRadioLabel = computed(() => {
  return middleCol.find(m => m.value === companyType.value)?.label || 'All';
});

const vClickOutside = {
  mounted(el, binding) {
    el.clickOutsideEvent = (e) => { if (!(el === e.target || el.contains(e.target))) binding.value(); };
    document.addEventListener("click", el.clickOutsideEvent);
  },
  unmounted(el) { document.removeEventListener("click", el.clickOutsideEvent); }
};
</script>

<style scoped>
.filter-row {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
  align-items: stretch;
  padding: 18px 24px;
  background: #f8f9fc;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(124,77,255,0.06);
  margin-top: 20px;
}

/* THE PILL STYLE */
.filter-pill {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  min-width: 180px;
  background: #fff;
  border: 1.5px solid #e2e8f0;
  border-radius: 16px;
  padding: 10px 18px;
  margin-bottom: 0;
  transition: 0.2s;
  cursor: pointer;
  user-select: none;
  box-shadow: 0 1px 4px rgba(124,77,255,0.04);
}

.filter-pill:hover { border-color: #7c4dff; }
.filter-pill.is-active { border-color: #7c4dff; background: #f5f3ff; }

.pill-label {
  font-size: 11px;
  font-weight: 800;
  color: #94a3b8;
  text-transform: uppercase;
  margin-bottom: 6px;
  padding-bottom: 6px;
  border-bottom: 1px solid #e2e8f0;
}

.is-active .pill-label { color: #7c4dff; border-bottom-color: #c4b5fd; }

.pill-trigger {
  display: flex;
  align-items: center;
  gap: 8px;
}

.pill-value {
  font-size: 15px;
  font-weight: 600;
  color: #1e293b;
}

.pill-count {
  background: #7c4dff;
  color: #fff;
  font-size: 11px;
  padding: 2px 7px;
  border-radius: 10px;
}

.simple-label {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* DROPDOWN MENU */
.pill-dropdown {
  position: absolute;
  top: 110%;
  left: 0;
  min-width: 200px;
  background: white;
  border: 1px solid #7c4dff;
  border-radius: 12px;
  box-shadow: 0 8px 20px rgba(124,77,255,0.08);
  z-index: 100;
  padding: 12px;
  display: flex;
  flex-direction: column;
}

.drop-item {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 12px 18px;
  font-size: 15px;
  border-radius: 8px;
  cursor: pointer;
  background: #fff;
  transition: background 0.1s;
}

.drop-item input[type="checkbox"], .drop-item input[type="radio"] {
  margin-right: 8px;
  width: 18px;
  height: 18px;
  accent-color: #7c4dff;
  vertical-align: middle;
}

.drop-item:hover { background: #f5f3ff; }

input[type="checkbox"], input[type="radio"] {
  accent-color: #7c4dff;
}

.chevron {
  width: 6px;
  height: 6px;
  border-right: 2px solid #94a3b8;
  border-bottom: 2px solid #94a3b8;
  transform: rotate(45deg);
  transition: 0.2s;
}

.chevron.open { transform: rotate(-135deg) translateY(-2px); }

@media (max-width: 900px) {
  .filter-row { gap: 12px; padding: 10px; }
  .filter-pill { min-width: 120px; padding: 8px 10px; }
  .pill-dropdown { min-width: 140px; padding: 8px; }
  .drop-item { font-size: 12px; padding: 8px 10px; }
}

@media (prefers-color-scheme: dark) {
  .filter-row {
    background: #181a20;
  }
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
  .pill-count {
    background: #a78bfa;
    color: #23263a;
  }
  .pill-dropdown {
    background: #23263a;
    border-color: #a78bfa;
    color: #e0e6f5;
    box-shadow: 0 10px 25px rgba(0,0,0,0.7);
  }
  .drop-item {
    background: #23263a;
    color: #e0e6f5;
  }
  .drop-item:hover {
    background: #2d2e4a;
    color: #a78bfa;
  }
  .drop-item input[type="checkbox"], .drop-item input[type="radio"] {
    accent-color: #a78bfa;
    background: #23263a;
    color: #e0e6f5;
  }
  .chevron {
    border-right-color: #a3aed6;
    border-bottom-color: #a3aed6;
  }
  .chevron.open {
    border-right-color: #a78bfa;
    border-bottom-color: #a78bfa;
  }
}
</style>