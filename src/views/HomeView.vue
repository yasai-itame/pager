<template>
  <!-- <div class="home">
    <HelloWorld msg="Welcome to Your Vue.js + TypeScript App"/>
  </div> -->
  <div class="bg-white py-6 sm:py-8 lg:py-12">
    <div class="max-w-screen-2xl px-4 md:px-8 mx-auto">
      <div class="grid sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 md:gap-6 xl:gap-8">
        <div v-for="result in results" :key="result.event_id" class="flex flex-col bg-white border rounded-lg overflow-hidden">
          <a :href="result.event_url" class="group h-48 md:h-64 block bg-gray-100" target="_blank">
            <div class="flex flex-wrap items-center h-full justify-center">
              <img src="@/assets/connpass_logo_1.png" loading="lazy" alt="" class="max-w-full h-auto" />
            </div>
          </a>
          <div class="flex flex-col flex-1 p-4 sm:p-6">
            <h2 class="text-gray-800 text-lg font-semibold mb-2">
              <a :href="result.event_url" class="hover:text-indigo-500 active:text-indigo-600 transition duration-100" target="_blank">{{result.title}}</a>
            </h2>
            <p class="text-gray-500 mb-8">{{result.catch}}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
  <section class="text-black bg-white body-font">
  <div class="container px-5 py-24 mx-auto">
    <div class="flex lg:w-2/3 w-full sm:flex-row flex-col mx-auto px-8 sm:px-0 items-end sm:space-x-4 sm:space-y-0 space-y-4">
      <div class="relative sm:mb-0 flex-grow w-full">
        <input type="text" v-model="search" class="w-full bg-white bg-opacity-40 rounded border border-gray-700 focus:border-indigo-500 focus:ring-2 focus:ring-indigo-900 focus:bg-transparent text-base outline-none text-black py-1 px-3 leading-8 transition-colors duration-200 ease-in-out">
      </div>
      <button @click="callApi(1)" class="text-white bg-indigo-500 border-0 py-2 px-8 focus:outline-none hover:bg-indigo-600 rounded text-lg">Search</button>
    </div>
  </div>
</section>
  <p>Search Keyword: {{search}}</p>
  <p>Result: {{pageInfo.total}}</p>
  <Pager :info="pageInfo" @current="callApi"></Pager>
</template>

<script lang="ts">
export default {
  name: 'HomeView'
}
</script>

<script setup lang="ts">
import { ref, reactive, onMounted } from "vue"
import axios from 'axios'
import jsonpAdapter from 'axios-jsonp'
import Pager from '@/components/Pager.vue'

let pageInfo = reactive({
  pagerStart: 1,
  pagerEnd: 0,
  current: 1,
  total: 0
})

let search = ref('css')

let results = ref()

let callApi = (pagerStart) => {
  axios({
    url: `https://connpass.com/api/v1/event/?keyword=${search.value}&start=${pagerStart}`,
    adapter: jsonpAdapter
  }).then((response) => {
    results.value = [...response.data.events]
    pageInfo.total = response.data.results_available
    pageInfo.pagerEnd = Math.floor(response.data.results_available / 10)
    pageInfo.current = response.data.results_start === 1 ? 1 : currentCalculation(response.data.results_start)
  })
}

let currentCalculation = ((num) => {
  return Math.floor(num / 10) + 1
})

onMounted(() => {
  // defaultが10件なので2ページ目はstartを11からにする
  callApi(pageInfo.pagerStart)
  /*
  2ページ目 .. 11
  3ページ目 .. 21
  7ページ目 .. 61
  */
})
</script>
