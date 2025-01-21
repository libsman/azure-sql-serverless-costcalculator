<template>
  <div class="space-y-4">
    <div class="space-y-4">
      <div>
        <label for="min-vcore-slider" class="block text-sm font-medium text-gray-700 mb-1">
          Min vCores: {{ minVCore }}
        </label>
        <input
          id="min-vcore-slider"
          type="range"
          :min="0.25"
          :max="maxVCore"
          :step="0.25"
          v-model.number="minVCore"
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
        />
      </div>

      <div>
        <label for="max-vcore-select" class="block text-sm font-medium text-gray-700 mb-1">
          Max vCores: {{ maxVCore }}
        </label>
        <select
          id="max-vcore-select"
          v-model.number="maxVCore"
          class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md"
        >
          <option v-for="vCore in vCoreOptions" :key="vCore" :value="vCore">
            {{ vCore }}
          </option>
        </select>
      </div>

      <div>
        <label for="hours-slider" class="block text-sm font-medium text-gray-700 mb-1">
          Aktive Stunden: {{ activeHours }}
        </label>
        <input
          id="hours-slider"
          type="range"
          min="1"
          max="24"
          step="1"
          v-model.number="activeHours"
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
        />
      </div>

      <div class="flex items-center">
        <div class="relative inline-block w-10 mr-2 align-middle select-none">
          <input
            id="auto-pause"
            type="checkbox"
            v-model="autoPauseOn"
            class="toggle-checkbox absolute block w-6 h-6 rounded-full bg-white border-4 appearance-none cursor-pointer"
          />
          <label for="auto-pause" class="toggle-label block overflow-hidden h-6 rounded-full bg-gray-300 cursor-pointer"></label>
        </div>
        <label for="auto-pause" class="text-sm text-gray-700 flex items-center">
          Auto-Pause {{ autoPauseOn ? 'An' : 'Aus' }}
          <span class="ml-1 group relative">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-gray-400 cursor-help" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            <span class="absolute bottom-full left-1/2 transform -translate-x-1/2 bg-gray-800 text-white text-xs rounded py-1 px-2 whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity duration-200 ease-in-out">
              Auto-Pause wird nach einer Stunde Inaktivität aktiviert
            </span>
          </span>
        </label>
      </div>

      <div>
        <label for="cost-factor" class="block text-sm font-medium text-gray-700 mb-1">
          Kostenfaktor pro Sekunde
        </label>
        <input
          id="cost-factor"
          type="number"
          v-model.number="costFactorPerSecond"
          step="0.0000001"
          min="0"
          class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
        />
      </div>

      <div class="text-xl sm:text-2xl font-bold text-gray-800">
        Geschätzte monatliche Kosten: €{{ minCost.toFixed(2) }} - €{{ maxCost.toFixed(2) }}
      </div>

      <CostChart
        :minVCore="minVCore"
        :maxVCore="maxVCore"
        :activeHours="activeHours"
        :autoPauseOn="autoPauseOn"
        :costFactorPerSecond="costFactorPerSecond"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import CostChart from './CostChart.vue'

const vCoreOptions = [1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 24, 32, 40, 48, 56, 64, 72, 80]
const minVCore = ref(0.5)
const maxVCore = ref(2)
const activeHours = ref(12)
const autoPauseOn = ref(true)
const costFactorPerSecond = ref(0.000149)

watch(maxVCore, (newValue) => {
  const minAllowed = Math.max(0.25, newValue / 8)
  minVCore.value = Math.min(Math.max(minVCore.value, minAllowed), newValue)
})

watch(minVCore, (newValue) => {
  if (newValue > maxVCore.value) {
    maxVCore.value = vCoreOptions.find(option => option >= newValue) || vCoreOptions[vCoreOptions.length - 1]
  }
})

const computeCost = (minVCore, maxVCore, activeHours, autoPauseOn, factorPerSecond) => {
  const costPerHour = factorPerSecond * 3600
  const activeCost = costPerHour * maxVCore * activeHours
  
  let inactiveCost = 0
  if (!autoPauseOn) {
    inactiveCost = costPerHour * minVCore * (24 - activeHours)
  } else if (activeHours < 24) {
    inactiveCost = costPerHour * minVCore * 1
  }

  return (activeCost + inactiveCost) * 30 // Monthly cost
}

const minCost = computed(() => 
  computeCost(minVCore.value, minVCore.value, activeHours.value, autoPauseOn.value, costFactorPerSecond.value)
)

const maxCost = computed(() => 
  computeCost(minVCore.value, maxVCore.value, activeHours.value, autoPauseOn.value, costFactorPerSecond.value)
)
</script>

<style scoped>
.toggle-checkbox:checked {
  @apply right-0 border-green-400;
}
.toggle-checkbox:checked + .toggle-label {
  @apply bg-green-400;
}
.group:hover .group-hover\:opacity-100 {
  opacity: 1;
}
</style>

