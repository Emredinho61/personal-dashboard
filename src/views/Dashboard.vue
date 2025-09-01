<template>
    <div class="card">
        <h3>Coming soon</h3>
        <div class="card flex justify-center">
            <SelectButton v-model="timeseriesvalue" :options="timeseriesoptions" />
        </div>
        <div class="p-4">
            <v-chart class="h-96 w-full" :option="option" theme="darkblue" />
        </div>
        IBM Sample:
        <pre>{{ testfetch }}</pre>
    </div>
</template>

<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';

const times = ref([]);
const closes = ref([]);
const option = ref({});
const testfetch = ref({});

const timeseriesvalue = ref('daily');
const timeseriesoptions = ref(['intraday', 'daily', 'weekly', 'monthly', 'yearly']);

function toGermanTime(isoString) {
    const date = new Date(isoString.replace(' ', 'T') + 'Z');
    return new Intl.DateTimeFormat('de-DE', {
        timeZone: 'Europe/Berlin',
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit'
    }).format(date);
}

function testfetchalphavantage() {
    axios
        .get('https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=IBM&interval=5min&apikey=demo')
        .then((response) => {
            testfetch.value = response.data;

            const rawData = response.data['Time Series (5min)'];
            if (!rawData) return;

            const t = [];
            const c = [];

            Object.entries(rawData)
                .sort(([a], [b]) => new Date(a) - new Date(b))
                .forEach(([time, values]) => {
                    t.push(toGermanTime(time));
                    c.push(Number(values['4. close']));
                });

            times.value = t;
            closes.value = c;

            option.value = {
                title: { text: 'IBM Intraday Prices (German Time)' },
                tooltip: { trigger: 'axis' },
                dataZoom: [
                    {
                        show: true,
                        realtime: true,
                        xAxisIndex: [0, 1]
                    },
                    {
                        type: 'inside',
                        realtime: true,
                        xAxisIndex: [0, 1]
                    }
                ],
                xAxis: { type: 'category', data: times.value },
                yAxis: { type: 'value', min: 'dataMin' },
                series: [
                    {
                        data: closes.value,
                        type: 'line',
                        smooth: true
                    }
                ]
            };
        })
        .catch((error) => {
            console.log(error);
        });
}

onMounted(() => {
    testfetchalphavantage();
});
</script>
