<template>
  <div class="p-6 space-y-4 relative">
    <div class="absolute top-0 right-0 p-2 text-sm opacity-80">Nama: Anam Badrus Salam<br/>NIM: 043079096</div>

    <h1 class="text-2xl font-bold">Weather Data (Jakarta)</h1>

    <button @click="fetchWeather" class="bg-blue-600 text-white px-4 py-2 rounded shadow hover:bg-blue-700">Load Data</button>

    <table class="w-full border rounded overflow-hidden shadow-sm">
      <thead class="bg-gray-100">
        <tr>
          <th class="border px-6 py-3">Time</th>
          <th class="border px-6 py-3 text-right">Temperature (°C)</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(t, index) in paginatedData" :key="index" class="hover:bg-gray-50">
          <td class="border px-6 py-2">{{ t.time }}</td>
          <td class="border px-6 py-2 text-right">{{ t.temp }}</td>
        </tr>
      </tbody>
    </table>

    <div class="flex items-center space-x-2">
      <span class="text-sm text-gray-600 mr-4">
        Showing {{ (page - 1) * perPage + 1 }} -
        {{ Math.min(page * perPage, combined.length) }}
        of {{ combined.length }}
      </span>
      <button
        @click="prevPage"
        :disabled="page === 1"
        class="px-3 py-1 border rounded disabled:opacity-40"
      >Prev</button>

      <span>Page {{ page }} / {{ totalPages }}</span>

      <button
        @click="nextPage"
        :disabled="page === totalPages"
        class="px-3 py-1 border rounded disabled:opacity-40"
      >Next</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  setup() {
    const weather = ref({ time: [] as string[], temperature: [] as number[] })

    const page = ref(1)
    const perPage = 10

    const fetchWeather = async () => {
      const res = await fetch(
        'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m'
      )
      const data = await res.json()

      weather.value.time = data.hourly.time
      weather.value.temperature = data.hourly.temperature_2m
      page.value = 1
    }

    const combined = computed(() =>
      weather.value.time.map((t, i) => {
        const date = new Date(t)
        const formatted = date.toLocaleString('id-ID', {
          day: '2-digit', month: 'short', year: 'numeric', hour: '2-digit', minute: '2-digit'
        })
        return { time: formatted, temp: weather.value.temperature[i] }
      })
    )

    const totalPages = computed(() => Math.ceil(combined.value.length / perPage))

    const paginatedData = computed(() => {
      const start = (page.value - 1) * perPage
      return combined.value.slice(start, start + perPage)
    })

    const nextPage = () => {
      if (page.value < totalPages.value) page.value++
    }

    const prevPage = () => {
      if (page.value > 1) page.value--
    }

    return { weather, fetchWeather, paginatedData, page, totalPages, nextPage, prevPage, perPage, combined }
  }
})
</script>

<style>
table { border-collapse: collapse; }
th, td { text-align: left; }
</style>
