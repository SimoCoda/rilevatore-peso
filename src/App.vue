<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const weightChartEl = ref(null);
const weightChart = shallowRef(null);
const weightInput = ref();
const currentWeight = computed(() => {
  return weights.value.sort((a,b) => b.date - a.date)[0] || {weight: 0}
});
const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime()
  })
}
watch(weights, newWeights => {
  const ws = [...newWeights];

  if(weightChart.value){
    weightChart.value.data.labels = ws.sort((a,b) => a.date - b.date).map(w => new Date(w.date).toLocaleDateString()).slice(-7);
    weightChart.value.data.datasets[0].data = ws.sort((a,b) => a.date - b.date).map(w => w.weight).slice(-7);
    weightChart.value.update();
    return;
  }
  
  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type:"line",
      data: {
        labels: ws.sort((a,b) => a.date - b.date).map(w => new Date(w.date).toLocaleDateString()),
        datasets: [
          {
            label: "Weight",
            data: ws.sort((a,b) => a.date - b.date).map(w => w.weight),
            backgroundColor: "rgba(81, 162, 255, .2)",
            borderColor: "rgba(81, 162, 255, 1)",
            borderWidth: 1,
            fill: true,
          }
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
      }
    })
  })
}, {deep: true})
</script>

<template>
  <main class="flex flex-col gap-10 items-center">
    <h1 class="text-2xl font-bold">Weight Tracker</h1>
    <div class="flex flex-col gap-1 w-40 h-40 rounded-[50%] bg-white border-blue-400 border-3 text-black items-center justify-center">
      <span class="text-xl font-medium">{{ currentWeight.weight }} kg</span>
      <small class="italic">Current weight (kg)</small>
    </div>
    <form @submit.prevent="addWeight" class="flex">
      <input type="number" placeholder="60.0" step="0.1" v-model="weightInput" class="bg-white text-black rounded-l-lg px-2 border-2 border-blue-400 border-r-0 focus-visible:outline-0 focus:border-blue-500" keydown.enter="addWeight" />
      <input type="submit" value="Add weight" class="bg-blue-500 text-white p-2 rounded-r-2xl hover:bg-blue-600 cursor-pointer" />
    </form>
    <div v-if="weights && weights.length > 0" class="w-full flex flex-col gap-8">
        <h2 class="text-2xl font-bold">Last 7 days</h2>
        <div class="w-full bg-white p-2 rounded-xl shadow-md shadow-gray-400">
          <canvas ref="weightChartEl" class=""></canvas>
        </div>
        <div id="weight-history" class="flex flex-col gap-4">
          <h2 class="text-2xl font-bold">Weight History</h2>
          <ul class="flex flex-col">
            <li v-for="weight in weights" :key="weight.date" class="flex justify-center items-center gap-6 odd:bg-gray-500 rounded-lg py-1">
              <span>
                {{  weight.weight }} kg
              </span>
              <small class="italic">
                {{ new Date(weight.date).toLocaleDateString() }}
              </small>
            </li>
          </ul>
        </div>
    </div>
  </main>
</template>
