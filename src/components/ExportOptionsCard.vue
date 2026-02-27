<template>
  <div class="export-options-heading">Export Options</div>
  <div class="export-system">
    <div class="filter-row">
      <div class="filter-pill">
        <span class="pill-label">Format</span>
        <select v-model="outputFormat" class="pill-select">
          <option v-for="fmt in ['Web Table', 'Spreadsheet', 'PDF', 'Text', 'XML']" :key="fmt">{{ fmt }}</option>
        </select>
      </div>

      <div class="filter-pill">
        <span class="pill-label">Font</span>
        <select v-model="fontSize" class="pill-select">
          <option v-for="size in fontSizes" :key="size" :value="size">{{ size }}px</option>
        </select>
      </div>

      <label v-if="outputFormat === 'PDF'" class="filter-pill checkbox-pill" :class="{ 'is-active': landscape }">
        <input type="checkbox" v-model="landscape" />
        <span class="pill-value">Landscape</span>
      </label>
    </div>

    <div class="action-row">
      <div class="segmented-control">
        <label v-for="opt in ['No Action', 'Fax', 'Email']" :key="opt" class="segment" :class="{ 'is-active': outputType === opt }">
          <input type="radio" v-model="outputType" :value="opt" />
          {{ opt }}
        </label>
      </div>

      <div v-if="outputType !== 'No Action'" class="dynamic-inputs">
        <div class="filter-pill input-pill">
          <span class="pill-label">{{ outputType === 'Fax' ? 'Number' : 'To' }}</span>
          <input 
            type="text" 
            v-model="contactValue" 
            :placeholder="outputType === 'Fax' ? 'Fax Number' : 'Email Address'" 
          />
        </div>

        <label v-if="outputType === 'Fax'" class="filter-pill checkbox-pill" :class="{ 'is-active': faxLandscape }">
          <input type="checkbox" v-model="faxLandscape" />
          <span class="pill-value">Fax Landscape</span>
        </label>
      </div>
    </div>

    <div v-if="outputType !== 'No Action'" class="message-container">
      <span class="pill-label">Message</span>
      <textarea 
        v-model="faxEmailMessage" 
        placeholder="Enter fax/email message..." 
        rows="1"
        @input="autoExpand"
      ></textarea>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const outputFormat = ref('Web Table');
const fontSize = ref(11);
const fontSizes = [8,9,10,11,12,13,14,15,16];
const landscape = ref(false);
const outputType = ref('No Action');
const faxNumber = ref('');
const emailTo = ref('');
const faxLandscape = ref(false);
const faxEmailMessage = ref('');

// Computed to map contact value based on selection
const contactValue = computed({
  get: () => outputType.value === 'Fax' ? faxNumber.value : emailTo.value,
  set: (val) => {
    if (outputType.value === 'Fax') faxNumber.value = val;
    else emailTo.value = val;
  }
});

const autoExpand = (e) => {
  e.target.style.height = 'inherit';
  e.target.style.height = `${e.target.scrollHeight}px`;
};
</script>

<style scoped>
.export-system {
  display: flex;
  flex-direction: column;
  gap: 12px;
  background: #f8f9fc;
  padding: 16px;
  border-radius: 12px;
  margin-top: 20px;
}

.filter-row, .action-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}
.filter-pill.checkbox-pill input{
width: unset;
}

/* REUSING YOUR PILL THEME */
.filter-pill {
  display: flex;
  align-items: center;
  background: #fff;
  border: 1.5px solid #e2e8f0;
  border-radius: 8px;
  padding: 4px 12px;
  height: 36px;
}

.pill-label {
  font-size: 10px;
  font-weight: 800;
  color: #94a3b8;
  text-transform: uppercase;
  margin-right: 8px;
  padding-right: 8px;
  border-right: 1px solid #e2e8f0;
}

.pill-select, .input-pill input {
  border: none;
  background: transparent;
  font-size: 13px;
  font-weight: 600;
  color: #1e293b;
  outline: none;
}

.checkbox-pill { cursor: pointer; }
.checkbox-pill.is-active { border-color: #faa643; background: #fff8ef; }
.checkbox-pill input { margin-right: 8px; accent-color: #faa643; }

/* SEGMENTED TOGGLE (Better than separate Radios) */
.segmented-control {
  display: flex;
  background: #e2e8f0;
  padding: 3px;
  border-radius: 8px;
}

.segment {
  padding: 5px 15px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
  border-radius: 6px;
  color: #64748b;
  transition: 0.2s;
}

.segment input { display: none; }
.segment.is-active { background: #fff; color: #faa643; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }

/* MESSAGE AREA */
.message-container {
  display: flex;
  flex-direction: column;
  background: #fff;
  border: 1.5px solid #e2e8f0;
  border-radius: 8px;
  padding: 8px 12px;
}

.message-container textarea {
  border: none;
  outline: none;
  font-size: 13px;
  color: #1e293b;
  resize: none;
  margin-top: 4px;
  width: 100%;
}

.dynamic-inputs { display: flex; gap: 8px; }

@media (max-width: 600px) {
  .dynamic-inputs { width: 100%; }
  .input-pill { flex: 1; }
}

@media (prefers-color-scheme: dark) {
  .export-system {
    background: #181a20;
  }
  .filter-pill {
    background: #23263a;
    border-color: #353a50;
    color: #e0e6f5;
  }
  .filter-pill.is-active, .checkbox-pill.is-active {
    border-color: #faa643;
    background: #23204a;
  }
  .pill-label {
    color: #a3aed6;
    border-right-color: #353a50;
  }
  .is-active .pill-label {
    color: #faa643;
    border-right-color: #faa643;
  }
  .pill-select, .input-pill input {
    background: transparent;
    color: #e0e6f5;
  }
  .pill-select option {
    background: #23263a;
    color: #e0e6f5;
  }
  .checkbox-pill input {
    accent-color: #faa643;
  }
  .segmented-control {
    background: #23263a;
  }
  .segment {
    color: #a3aed6;
  }
  .segment.is-active {
    background: #23204a;
    color: #faa643;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  }
  .message-container {
    background: #23263a;
    border-color: #faa643;
  }
  .message-container textarea {
    background: transparent;
    color: #e0e6f5;
  }
}
</style>
<style scoped>
.export-options-heading {
  font-size: 1.4rem;
  font-weight: 700;
  color: #147393;
  margin-bottom: 0.5rem;
  margin-top: 0.5rem;
  letter-spacing: 0.01em;
  font-family: 'Inter', -apple-system, sans-serif;
}
</style>