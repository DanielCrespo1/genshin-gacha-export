<template>
  <el-button class="focus:outline-none" :disabled="!allowClick()" plain size="small" @click="fetchData" :loading="state.status === 'loading'">{{state.status === 'init' ? 'Load Data': 'Update Data'}}</el-button>
  <el-button @click="saveExcel" class="focus:outline-none" :disabled="!state.data" size="small" type="success" plain>Export to Excel</el-button>
  <p class="text-gray-400 my-2 text-xs">{{hint}}</p>
  <div v-if="detail" class="flex gap-4 flex-wrap justify-between cont">
    <div class="flex-grow flex-shrink-0 mb-4 w-64" v-for="(item, i) of detail" :key="i">
      <p class="text-center text-gray-600 my-2">{{typeMap.get(item[0])}}</p>
      <pie-chart :data="item" :typeMap="typeMap"></pie-chart>
      <gacha-detail :data="item" :typeMap="typeMap"></gacha-detail>
    </div>
    <i class="flex-grow flex-shrink-0 w-64 cat:hidden" v-for="i in 2" :key="i"></i>
  </div>
</template>

<script setup>
const { ipcRenderer } = require('electron')
import { reactive, computed } from 'vue'
import PieChart from './components/PieChart.vue'
import GachaDetail from './components/GachaDetail.vue'
import gachaDetail from './gachaDetail'
import { version } from '../../package.json'

const state = reactive({
  status: 'init',
  log: '',
  data: null
})

const allowClick = () => {
  if (!state.data) return true
  if (Date.now() - state.data.time < 1000 * 60) {
    return false
  }
  return true
}

const hint = computed(() => {
  if (state.status === 'init') {
    return 'Please login to Genshin, open up your wish history and click on "Load Data"'
  } else if (state.status === 'loaded') {
    return `Last Updated：${new Date(state.data.time).toLocaleString()}`
  } else if (state.status === 'loading') {
    return state.log
  } else if (state.status === 'failed') {
    return state.log + ' - Failed'
  }
  return ''
})

const detail = computed(() => {
  if (state.data) {
    return gachaDetail(state.data.result)
  }
})

const typeMap = computed(() => state.data.typeMap)

const fetchData = async () => {
  state.status = 'loading'
  const data = await ipcRenderer.invoke('FETCH_DATA')
  if (data) {
    state.data = data
    state.status = 'loaded'
  } else {
    state.status = 'failed'
  }
}

const readData = async () => {
  const data = await ipcRenderer.invoke('READ_DATA')
  if (data) {
    state.data = data
    state.status = 'loaded'
  }
}

const saveExcel = async () => {
  await ipcRenderer.invoke('SAVE_EXCEL')
}

readData()

ipcRenderer.on('LOAD_DATA_STATUS', (event, message) => {
  state.log = message
})

ipcRenderer.on('ERROR', (event, err) => {
  console.error(err)
})

document.title = `Genshin Gacha Exporter - v${version}`
console.log('http://music.163.com/song?id=33913985')
</script>
