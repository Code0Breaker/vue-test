<template>
    <div>
        <h1>Конвертация валют</h1>
        <div class="converter">
            <div class="row">
                <select v-model="currencyFrom">
                    <option value="USD">USD</option>
                    <option value="EUR">EUR</option>
                    <option value="RUB">RUB</option>
                    <option value="BRL">BRL</option>
                    <option value="IDR">IDR</option>
                    <option value="KZT">KZT</option>
                </select>
                <input type="number" v-model.number="amountFrom" @input="convertFrom" />
            </div>
            <div class="row">
                <select v-model="currencyTo">
                    <option value="USD">USD</option>
                    <option value="EUR">EUR</option>
                    <option value="RUB">RUB</option>
                    <option value="BRL">BRL</option>
                    <option value="IDR">IDR</option>
                    <option value="KZT">KZT</option>
                </select>
                <input type="number" v-model.number="amountTo" @input="convertTo" />
            </div>
        </div>
        <div v-if="error" class="error">{{ error }}</div>
    </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, nextTick } from 'vue';

interface Rates {
    [key: string]: number;
}

const currencyFrom = ref<string>('USD');
const currencyTo = ref<string>('RUB');
const amountFrom = ref<number>(0);
const amountTo = ref<number>(0);
const rates = ref<Rates>({});
const error = ref<string | null>(null);

const fetchRates = async (): Promise<void> => {
    try {
        const response = await fetch('https://status.neuralgeneration.com/api/currency');
        if (!response.ok) {
            throw new Error('Failed to fetch exchange rates');
        }
        const data = await response.json();
        rates.value = data;
    } catch (err) {
        error.value = 'Ошибка при загрузке курсов валют';
        console.error(err);
    }
};

const convertFrom = async (): Promise<void> => {
    await nextTick();
    const key = `${currencyFrom.value.toLowerCase()}-${currencyTo.value.toLowerCase()}`;
    if (!rates.value[key]) {
        amountTo.value = 0;
        return;
    }
    const rate = rates.value[key];
    amountTo.value = parseFloat((amountFrom.value * rate).toFixed(2));
};

const convertTo = async (): Promise<void> => {
    await nextTick();
    const key = `${currencyTo.value.toLowerCase()}-${currencyFrom.value.toLowerCase()}`;
    if (!rates.value[key]) {
        amountFrom.value = 0;
        return;
    }
    const rate = rates.value[key];
    amountFrom.value = parseFloat((amountTo.value * rate).toFixed(2));
};

watch([currencyFrom, currencyTo], async () => {
    await nextTick();
    if (amountFrom.value !== 0) {
        convertFrom();
    } else if (amountTo.value !== 0) {
        convertTo();
    }
});

onMounted(fetchRates);
</script>

<style>
.converter {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.row {
    display: flex;
    gap: 10px;
    align-items: center;
}

.error {
    color: red;
}
</style>
