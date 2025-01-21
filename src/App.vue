<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <h1 class="text-2xl sm:text-3xl font-bold mb-4 text-center text-gray-800">Azure SQL Serverless Cost Calculator</h1>
        <p class="text-sm text-gray-500 text-center mb-6">Hinweis: Diese Berechnungen sind inoffizielle Schätzungen und keine offiziellen Microsoft-Angaben. Ohne Gewähr.</p>
        <AzureSQLCalculator />
      </div>
    </div>
  </div>
</template>

<script setup>
import AzureSQLCalculator from './components/AzureSQLCalculator.vue'
</script>

