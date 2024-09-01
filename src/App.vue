<template>
  <header>
    <nav>
      <router-link to="/">Главная</router-link>
      <router-link to="/convert">Конвертация</router-link>
    </nav>
    <div>
      <label for="base-currency">Основная валюта:</label>
      <select id="base-currency" v-model="baseCurrency" @change="updateBaseCurrency">
        <option value="USD">USD</option>
        <option value="EUR">EUR</option>
        <option value="RUB">RUB</option>
      </select>
    </div>
  </header>
  <main>
    <RouterView />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { RouterView } from 'vue-router';

const baseCurrency = ref<string>(localStorage.getItem('baseCurrency') || 'USD');

const updateBaseCurrency = (): void => {
  localStorage.setItem('baseCurrency', baseCurrency.value);
  window.location.reload(); // Reload to update rates
};
</script>

<style>
header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: #f4f4f4;
  border-bottom: 1px solid #ccc;
}

nav a {
  margin: 0 10px;
  text-decoration: none;
  color: #333;
}

nav a.router-link-active {
  font-weight: bold;
}
</style>
