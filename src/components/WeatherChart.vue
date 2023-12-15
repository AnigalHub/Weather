<template>
        <LineChart ref="chart" v-if="showChart && type === 'Line' && localData"
                   :options="chartOptions"
                   :data="localData"
        />
        <BarChart ref="chart" v-else-if="showChart && type === 'Bar' && localData"
                  :options="chartOptions"
                  :data="localData"
        />
</template>

<script>
    import {
        Chart as ChartJS,
        CategoryScale,
        LinearScale,
        PointElement,
        LineElement,
        BarElement,
        Title,
        Tooltip,
        Legend,
    } from 'chart.js'
    import { Line, Bar } from 'vue-chartjs'

    ChartJS.register(
        CategoryScale,
        LinearScale,
        PointElement,
        LineElement,
        BarElement,
        Title,
        Tooltip,
        Legend
    )

    export default {
        name: "WeatherChart",
        components: { LineChart: Line, BarChart: Bar},
        props: {
            chartData: {
                type: Object,
                default: null,
            },
            chartOptions: Object,
            type: {
                type: String,
                default: "Line"
            }
        },
        data()  {
            return {
                showChart: false,
                localData: null,
            }
        },
        watch: {
            chartData:{
                immediate: true,
                handler(newVal, oldVal){
                    console.log("watcher", {newVal, oldVal})
                    this.localData = newVal;
                },
            },

        },
        created() {
            setTimeout(() => this.showChart = true, 100)
        },
    }
</script>

<style scoped>
</style>