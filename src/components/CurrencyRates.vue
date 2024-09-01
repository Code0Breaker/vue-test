<template>
    <div>
        <h1>Курсы валют (относительно {{ baseCurrency }})</h1>
        <ul>
            <li v-for="(rate, key) in displayRates" :key="key">
                1 {{ key.split('-')[0].toUpperCase() }} = {{ rate.toFixed(2) }} {{ key.split('-')[1].toUpperCase() }}
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

interface Rates {
    [key: string]: number;
}

const rates = ref<Rates>({});
const baseCurrency = ref<string>(localStorage.getItem('baseCurrency') || 'USD');

const fetchRates = async (): Promise<void> => {
    try {
        const response = await fetch('https://status.neuralgeneration.com/api/currency');
        if (!response.ok) {
            throw new Error('Failed to fetch exchange rates');
        }
        const data = await response.json();
        rates.value = data;
    } catch (err) {
        console.error('Ошибка при загрузке курсов валют:', err);
    }
};

const displayRates = computed(() => {
    const filteredRates: Rates = {};
    for (const [key, value] of Object.entries(rates.value)) {
        if (typeof key === 'string') {
            console.log('Processing key:', key);

            const [from, to] = key.split('-');
            if (from && to) {
                if (from === baseCurrency.value.toLowerCase()) {
                    filteredRates[`${from}-${to}`] = value;
                } else if (to === baseCurrency.value.toLowerCase()) {
                    filteredRates[`${to}-${from}`] = 1 / value;
                }
            }
        }
    }
    return filteredRates;
});


onMounted(fetchRates);
</script>
