<template>
  <nav class="isolate inline-flex -space-x-px rounded-md shadow-sm" aria-label="Pagination">
    <div v-for="range in ranges" :key="range.id">
      <template v-if="range.element === 'leftAllow'">
        <span @click="pagerAction(1)" class="relative inline-flex items-center rounded-l-md border border-gray-300 bg-white px-2 py-2 text-sm font-medium text-gray-500 hover:bg-gray-50 focus:z-20 cursor-pointer">
          <span class="sr-only">Previous</span>
          <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
            <path fill-rule="evenodd" d="M12.79 5.23a.75.75 0 01-.02 1.06L8.832 10l3.938 3.71a.75.75 0 11-1.04 1.08l-4.5-4.25a.75.75 0 010-1.08l4.5-4.25a.75.75 0 011.06.02z" clip-rule="evenodd" />
          </svg>
        </span>
      </template>
      <template v-if="range.element === 'dots'">
        <span class="relative inline-flex items-center border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700">...</span>
      </template>
      <template v-if="range.element !== 'leftAllow' && range.element !== 'dots' && range.element !== 'rightAllow'">
        <template v-if="range.element === current">
          <span aria-current="page" class="relative z-10 inline-flex items-center border border-indigo-500 bg-indigo-50 px-4 py-2 text-sm font-medium text-indigo-600 focus:z-20">{{range.element}}</span>
        </template>
        <template v-else>
          <span @click="pagerAction(range.element)" class="relative inline-flex items-center border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-500 hover:bg-gray-50 focus:z-20 cursor-pointer">{{range.element}}</span>
        </template>
      </template>
      <template v-if="range.element === 'rightAllow'">
        <span @click="pagerAction(lastArrow)" class="relative inline-flex items-center rounded-r-md border border-gray-300 bg-white px-2 py-2 text-sm font-medium text-gray-500 hover:bg-gray-50 focus:z-20 cursor-pointer">
          <span class="sr-only">Next</span>
          <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
            <path fill-rule="evenodd" d="M7.21 14.77a.75.75 0 01.02-1.06L11.168 10 7.23 6.29a.75.75 0 111.04-1.08l4.5 4.25a.75.75 0 010 1.08l-4.5 4.25a.75.75 0 01-1.06-.02z" clip-rule="evenodd" />
          </svg>
        </span>
      </template>
    </div>
  </nav>
</template>
<script lang="ts">
export default {
  name: 'Pager'
}
</script>

<script setup lang="ts">
import { defineProps, defineEmits, ref, toRefs, watch } from "vue"
const props = defineProps({
  info: {
    type: Object
  }
})

const { info } = toRefs(props)
const ranges = ref([])
let current = ref(0)
let lastArrow = ref(0)
watch(info, (newVal) => {
  if (newVal) {
    let obj = {}
    ranges.value = []

    current.value = Number(newVal.current)
    lastArrow.value = Number(newVal.pagerEnd)
    if (info.value.pagerEnd >= 8) {
      let plusCounter = 0
      for (let i = 0; i < 9; i++) {
        obj['id'] = i + 1

        if (i === 0) {
          obj['element'] = 'leftAllow'
        }

        // 1 2 3 4 5 ... 10
        if (current.value <= 4) {
          if (i === 6) {
            obj['element'] = 'dots'
          } else if (i === 7) {
            obj['element'] = info.value.pagerEnd
          } else if (i >= 1 && i <= 5) {
            obj['element'] = plusCounter + 1
            plusCounter++
          }
        }

        // 1 ... 4 5 6 ... 10
        if (current.value >= 5 && current.value <= info.value.pagerEnd - 3 && current.value !== info.value.pagerEnd) {
          if (i === 2 || i === 6) {
            obj['element'] = 'dots'
          } else if (i === 1 || i >= 3) {
            if (i === 1) {
              obj['element'] = 1
            } else if (i === 7) {
              obj['element'] = info.value.pagerEnd
            } else if (i === 3) {
              obj['element'] = current.value - 1
            } else if (i === 4) {
              obj['element'] = current.value
            } else if (i === 5) {
              obj['element'] = current.value + 1
            }
          }
        }

        // 1 ... 6 7 8 9 10
        if (current.value > info.value.pagerEnd - 3) {
          if (i === 1) {
            obj['element'] = 1
          } else if (i === 2) {
            obj['element'] = 'dots'
          } else if (i !== 0 && i !== 8) {
            if (current.value <= info.value.pagerEnd - 3) {
              obj['element'] = current.value - 1
            } else {
              obj['element'] = info.value.pagerEnd - 4 + plusCounter
              plusCounter++
            }
          }
        }

        if (i === 8) {
          obj['element'] = 'rightAllow'
        }

        ranges.value.push(Object.assign({}, obj))
      }
    } else {
      let pagerCount = info.value.pagerEnd
      if (info.value.pagerEnd === 0) return false
      let plusCounter = 0
      for (let i = 0; i <= pagerCount + 1; i++) {
        obj['id'] = i + 1
        if (i === 0) {
          obj['element'] = 'leftAllow'
        }
        if (i !== 0 && i !== pagerCount + 1) {
          obj['element'] = plusCounter + 1
          plusCounter++
        }
        if (i === pagerCount + 1) {
          obj['element'] = 'rightAllow'
        }
        // < 1 2 3 4 5 6 7 >

        ranges.value.push(Object.assign({}, obj))
      }
    }
  }
}, { deep: true })

const emit = defineEmits(
  ['current']
)

let pagerAction = ((pages) => {
  let page
  let count = 10
  if (pages !== 1) {
    let multiplication = pages - 1
    page = count * multiplication + 1
  } else {
    page = 1
  }
  emit('current', page)
})
</script>