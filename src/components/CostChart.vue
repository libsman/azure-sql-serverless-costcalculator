<template>
  <div ref="chartContainer" class="w-full h-64 sm:h-80">
    <canvas ref="chartRef"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import Chart from 'chart.js/auto'

const props = defineProps({
  minVCore: Number,
  maxVCore: Number,
  activeHours: Number,
  autoPauseOn: Boolean,
  costFactorPerSecond: Number
})

const chartRef = ref(null)
const chartContainer = ref(null)
let chart = null

const computeCost = (vCore, activeHours, autoPauseOn, factorPerSecond) => {
  const costPerHour = factorPerSecond * 3600
  const activeCost = costPerHour * vCore * activeHours
  
  let inactiveCost = 0
  if (!autoPauseOn) {
    inactiveCost = costPerHour * vCore * (24 - activeHours)
  } else if (activeHours < 24) {
    inactiveCost = costPerHour * vCore * 1
  }

  return (activeCost + inactiveCost) * 30 // Monthly cost
}

const generateVCoreValues = (maxVCore) => {
  const baseValues = [0.25, 0.5, 1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 24, 32, 40, 48, 56, 64, 72, 80]
  const values = baseValues.filter(v => v <= maxVCore)
  if (!values.includes(maxVCore)) {
    values.push(maxVCore)
  }
  return values.sort((a, b) => a - b)
}

const updateChart = () => {
  try {
    const maxVCore = props.maxVCore
    const vCores = generateVCoreValues(maxVCore)

    const minData = vCores.map(vc => ({
      x: vc,
      y: vc >= props.minVCore ? computeCost(props.minVCore, props.activeHours, props.autoPauseOn, props.costFactorPerSecond) : null
    })).filter(point => point.y !== null)

    const maxData = vCores.map(vc => ({
      x: vc,
      y: vc >= props.minVCore ? computeCost(vc, props.activeHours, props.autoPauseOn, props.costFactorPerSecond) : null
    })).filter(point => point.y !== null)

    if (minData.length === 0 || maxData.length === 0) {
      console.warn('No valid data points to display')
      return
    }

    const xAxisLabels = () => {
      if (maxVCore <= 4) return vCores
      if (maxVCore <= 20) return vCores.filter(v => v % 2 === 0 || v === maxVCore)
      return vCores.filter(v => v % 4 === 0 || v === maxVCore)
    }

    if (chart) {
      try {
        chart.data.datasets[0].data = minData
        chart.data.datasets[1].data = maxData
        chart.options.scales.x.max = maxVCore
        chart.options.scales.x.ticks.callback = (value) => xAxisLabels().includes(value) ? value : ''
        chart.update('none') // Add update mode to prevent animation issues
      } catch (error) {
        console.error('Error updating chart:', error)
      }
    } else {
      chart = new Chart(chartRef.value, {
        type: 'line',
        data: {
          datasets: [
            {
              label: 'Min Kosten',
              data: minData,
              borderColor: 'rgb(59, 130, 246)',
              backgroundColor: 'rgba(59, 130, 246, 0.1)',
              fill: '+1',
              tension: 0
            },
            {
              label: 'Max Kosten',
              data: maxData,
              borderColor: 'rgb(239, 68, 68)',
              backgroundColor: 'rgba(239, 68, 68, 0.1)',
              fill: false,
              tension: 0
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              type: 'linear',
              position: 'bottom',
              title: {
                display: true,
                text: 'vCores'
              },
              min: 0.25,
              max: maxVCore,
              ticks: {
                callback: function(value) {
                  return xAxisLabels().includes(value) ? value : ''
                }
              }
            },
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: 'Kosten (€)'
              }
            }
          },
          plugins: {
            legend: {
              display: true,
              position: 'top'
            }
          }
        }
      })
    }
  } catch (error) {
    console.error('Fehler beim Aktualisieren des Diagramms:', error)
  }
}

onMounted(() => {
  updateChart()
  
  const resizeObserver = new ResizeObserver(() => {
    if (chart) {
      try {
        chart.resize()
      } catch (error) {
        console.error('Error resizing chart:', error)
        // Reinitialize chart if resize fails
        chart.destroy()
        chart = null
        updateChart()
      }
    }
  })

  if (chartContainer.value) {
    resizeObserver.observe(chartContainer.value)
  }

  onUnmounted(() => {
    resizeObserver.disconnect()
    if (chart) {
      chart.destroy()
    }
  })
})

watch([() => props.minVCore, () => props.maxVCore, () => props.activeHours, () => props.autoPauseOn, () => props.costFactorPerSecond], () => {
  updateChart()
})
</script>

