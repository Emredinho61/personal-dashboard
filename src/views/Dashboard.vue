<template>
    <div class="card">
        <h3>Coming soon</h3>
        IBM Sample:
        {{testfetch}}
    </div>
</template>
<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';

const testfetch = ref({})

function testfetchalphavantage(){
    axios.get("https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=IBM&interval=5min&apikey=demo")
    .then((response) => {
        testfetch.value = response.data;
    })
    .catch((error) => {
        console.log(error);
    })
}

onMounted(() => {
    testfetchalphavantage();
})
</script>
