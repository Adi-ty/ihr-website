<script setup>
import ReactiveChart from './ReactiveChart.vue'
import { ref, computed, watch, onMounted } from 'vue'
import * as TimeSeriesAggregatedAlarmsDataModel from '@/plugins/models/TimeSeriesAggregatedAlarmsDataModel'
import * as AggregatedAlarmsUtils from '@/plugins/utils/AggregatedAlarmsUtils'

const props = defineProps({
  loading: {
    type: Boolean,
    required: true,
  },
  alarms: {
    type: Array,
  },
  aggregatedAttrsSelected: {
    type: Object,
  },
  countryName: {
    type: String
  },
  alarmTypeTitlesMap: {
    type: Object
  },
  networkName: {
    type: String
  }
})

const emits = defineEmits(['country-clicked', 'select-time'])

const layout = ref({
  xaxis: {
    title: 'Date',
    autorange: true
  },
  yaxis: {
    title: 'Number of Alarms',
    autorange: true
  },
  hovermode: 'closest',
  showlegend: true,
  legend: {
    x: 1,
    xanchor: 'top',
    y: 1
  },
})
const traces = ref([])

const noData = computed(() => {
  if (!props.loading && !traces.value.length) {
    return 'No data to show'
  } else if (!props.loading) {
    return false
  } else {
    return 'Loading...'
  }
})

const init = (alarms, aggregatedAttrsSelected, countryName, alarmTypeTitlesMap, networkName, isASGranularity) => {
  const aggregatedAttrsZipped = AggregatedAlarmsUtils.zipAggregatedAttrs(aggregatedAttrsSelected)
  const timeSeriesTraces = TimeSeriesAggregatedAlarmsDataModel.etl(alarms, aggregatedAttrsZipped, countryName, alarmTypeTitlesMap, networkName, isASGranularity)
  if (!timeSeriesTraces.length) {
    clearDataViz()
  } else {
    traces.value = timeSeriesTraces
  }
  if (layout.value.xaxis && layout.value.yaxis) {
    layout.value.xaxis.autorange = true
    layout.value.yaxis.autorange = true
  }
}

const clearDataViz = () => {
  traces.value = []
}

const plotlyClickedDataHandler = (val) => {
  emits('country-clicked', val)
}

const plotlySelectTimeHandler = (obj) => {
  emits('select-time', obj)
}

watch(() => props.alarms, () => {
  if (props.networkName) {
    init(props.alarms, props.aggregatedAttrsSelected, props.countryName, props.alarmTypeTitlesMap, props.networkName, true)
  } else {
    init(props.alarms, props.aggregatedAttrsSelected, props.countryName, props.alarmTypeTitlesMap)
  }
})

onMounted(() => {
  init(props.alarms, props.aggregatedAttrsSelected, props.countryName, props.alarmTypeTitlesMap)
})
</script>

<template>
  <div class="IHR_chart">
    <div class="IHR_disco-chart">
        <ReactiveChart :layout="layout" :traces="traces" :no-data="noData" @plotly-legend-click="plotlyClickedDataHandler" @plotly-time-filter="plotlySelectTimeHandler" />
    </div>
  </div>
</template>