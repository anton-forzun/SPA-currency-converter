<script setup>
import { ref,watch,onMounted } from "vue";

const selectCurrencyFrom = ref("");
const selectCurrencyTo = ref("");
const inputAmount = ref("");
const choiseCurrency = ref("");
const addCurrency = ref("");
const result = ref("");
const tableItems = ref(null);
const favoriteCurrency = ref("");
const currencyList = ref([]);
const popup = ref(false);


const ACCESS_KEY = "4EUgS9oUdVjkzMRhh2STGUfRPri50sPv";

async function ChoiseCurrency(source, currencies) {
  const data = await fetch(
    `https://api.apilayer.com/currency_data/live?source=${source}&currencies=USD,EUR,UAH,BTC,ETH`,
    {
      method: "GET",
      headers: {
        Accept: "application/json",
        apikey: ACCESS_KEY,
      },
    }
  );
  const res = await data.json();
  tableItems.value = res.quotes;
  
}
async function addCurrencyLive(source, currencies) {
  const data = await fetch(
    `https://api.apilayer.com/currency_data/live?source=${source}&currencies=${currencies}`,
    {
      method: "GET",
      headers: {
        Accept: "application/json",
        apikey: ACCESS_KEY,
      },
    }
  );
  const res = await data.json();
  addCurrency.value = res.quotes;
  console.log(res.quotes);
  console.log(addCurrency);
}

async function CurrencyApi(currencyForm, currencyTo, amount) {
  const data = await fetch(
    `https://api.apilayer.com/currency_data/convert?to=${currencyTo}&from=${currencyForm}&amount=${amount}`,
    {
      method: "GET",
      headers: {
        Accept: "application/json",
        apikey: ACCESS_KEY,
      },
    }
  );
  const res = await data.json();
  result.value = res;
 
}


const choiseCurrencyLive = () => {
  ChoiseCurrency(choiseCurrency.value);
};
const addCurrencyItem = () => {
  addCurrencyLive(choiseCurrency.value, addCurrency.value);
  currencyList.value.push(addCurrency);
  console.log(currencyList)
  popup.value = false;
};
const removeCurrencyItem = (listItem) => {
  currencyList.value = currencyList.value.filter((t) => t !== listItem);
};

const searchCurrency = () => {
  if (selectCurrencyFrom.value === "") {
    alert("Please choose currency from");
    inputAmount.value = "";
    return;
  }
  if (selectCurrencyTo.value === "") {
    alert("Please choose currency to");
    inputAmount.value = "";
    return;
  }
  if (inputAmount.value === "") {
    alert("Please enter the amount");
    return;
  }
  if (inputAmount.value > 10000) {
    alert("You have exeeded the currency limit,no more than 10000");
    return;
  }
  CurrencyApi(
    selectCurrencyFrom.value,
    selectCurrencyTo.value,
    inputAmount.value
  );
};

const refresh = () => {
  ChoiseCurrency(choiseCurrency.value);
};

watch(
  currencyList,(newVal) => {localStorage.setItem("currencyList", JSON.stringify(newVal));},{deep: true,});

  onMounted(() => {currencyList.value = JSON.parse(localStorage.getItem("currencyList")) || [];});
</script>

<template>
  <div class="container">
    <div class="container__form">
     
      <div class="form__currency">
        <select class="select" v-model="selectCurrencyFrom">
          <option disabled value="">Choise currency</option>
          <option value="USD">USD</option>
          <option value="EUR">EUR</option>
          <option value="UAH">UAH</option>
          <option value="GBP">GBP</option>
          <option value="BTC">BTC</option>
          <option value="ETH">ETH</option>
          <option value="XRP">XRP</option>
        </select>
        <select class="select" v-model="selectCurrencyTo">
          <option disabled value="">Choise currency</option>
          <option value="BTC">BTC</option>
          <option value="EUR">EUR</option>
          <option value="UAH">UAH</option>
          <option value="GBP">GBP</option>
          <option value="USD">USD</option>
          <option value="ETH">ETH</option>
          <option value="XRP">XRP</option>
        </select>

        <input
          class="input"
          v-model="inputAmount"
          @keyup="searchCurrency"
          placeholder="Input amount"
          type="text"
        />
      </div>
      <h2 v-if="result.result" class="result">
        RESULT:<span class="form__currency--to">{{
          result.result.toFixed(3)
        }}</span>
        <span class="form__currency--to">{{ result.query.to }}</span>
      </h2>
    </div>
    <div class="container__table">
      <div class="container__choise">
        <select
          @change="choiseCurrencyLive"
          class="select choise"
          v-model="choiseCurrency"
        >
          <option disabled value="">Choise currency</option>
          <option value="USD">USD</option>
          <option value="EUR">EUR</option>
          <option value="UAH">UAH</option>
        </select>
        <div class="button__container">
          <button class="button blue" @click="popup = true">Add currency</button>
          <button class="button" @click="refresh">Refresh</button>
        </div>
      </div>
      <div v-if="tableItems" class="table">
        <div
          v-for="item in Object.entries(tableItems)"
          :key="item"
          class="table__items"
        >
          <div class="items">
            <h3>{{ item[0].slice(3, 6) }}</h3>
            <h3>{{ item[1] }}</h3>
          </div>
        </div>
      </div>
      <div v-if="currencyList">
        <div class="table__items" v-for="listItem in currencyList" :key="listItem">
          <div class="items">
            <h3>{{listItem}}</h3>
          </div>
          <button class="button blue" @click="removeCurrencyItem(listItem)">delete</button>
        </div>
      </div>
    </div>
    <div v-if="popup" class="popup">
      <div class="popup__background"></div>
      <div class="popup__card">
        <select
          @change="addCurrencyItem"
          class="select choise popup__select"
          v-model="addCurrency"
        >
          <option disabled value="">Choise currency</option>
          <option value="BTC">BTC</option>
          <option value="EUR">EUR</option>
          <option value="UAH">UAH</option>
          <option value="GBP">GBP</option>
          <option value="USD">USD</option>
          <option value="ETH">ETH</option>
          <option value="XRP">XRP</option>
        </select>
      </div>
    </div>
  </div>
</template>

<style>

</style>