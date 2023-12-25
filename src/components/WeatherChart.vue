<template>
        <LineChart ref="chart" v-if="showChart && type === 'Line' && localData"
                   :options="chartOptions"
                   :data="localData"
        />
        <BarChart ref="chart" v-else-if="showChart && type === 'Bar' && localData"
                  :options="chartOptions"
                  :data="localData"
        />
        <DoughnutChart ref="chart" v-else-if="showChart && type === 'Doughnut' && localData"
                  :options="chartOptions"
                  :data="localData"
        />
        <BubbleChart ref="chart" v-else-if="showChart && type === 'Bubble' && localData"
                :options="chartOptions"
                :data="localData"
        />
        <PieChart ref="chart" v-else-if="showChart && type === 'Pie' && localData"
                  :options="chartOptions"
                  :data="localData"
        />
        <ScatterChart ref="chart" v-else-if="showChart && type === 'Scatter' && localData"
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
        ArcElement,
        BarElement,
        Title,
        Tooltip,
        Legend,
    } from 'chart.js'
    import { Line, Bar, Doughnut, Bubble, Pie, Scatter } from 'vue-chartjs'

    ChartJS.register(
        CategoryScale,
        LinearScale,
        PointElement,
        LineElement,
        ArcElement,
        BarElement,
        Title,
        Tooltip,
        Legend
    )

    export default {
        name: "WeatherChart",
        components: { LineChart: Line, BarChart: Bar, DoughnutChart:Doughnut, BubbleChart:Bubble, PieChart: Pie, ScatterChart:Scatter },
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
                handler(newVal){
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