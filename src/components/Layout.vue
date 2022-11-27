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
.container {
  display: flex;
  position: relative;
  align-items: center;
  flex-direction: column;
  color: black;
}
.container__form {
  padding: 40px 20px;
  border: 2px solid rgb(99, 116, 198);
  border-radius: 15px;
  box-shadow: rgba(99, 98, 98, 0.35) 0px 5px 15px;
}
.select {
  width: 100%;
  min-width: 15ch;
  max-width: 30ch;
  border: 1px solid var(--select-border);
  border-radius: 0.25em;
  padding: 0.25em 0.5em;
  font-size: 1.25rem;
  cursor: pointer;
  line-height: 1.1;
  background-color: #fff;
  background-image: linear-gradient(to top, #f9f9f9, #fff 33%);
}
.input {
  width: 100%;
  min-width: 15ch;
  max-width: 30ch;
  border: 1px solid var(--select-border);
  border-radius: 0.25em;
  padding: 0.25em 0.5em;
  font-size: 1.25rem;
  cursor: pointer;
  line-height: 1.1;
  background-color: #fff;
  background-image: linear-gradient(to top, #f9f9f9, #fff 33%);
}
.form__currency {
  display: flex;
  
  gap: 5px;
}
.form__currency--to {
  padding: 5px;
}
.container__table {
  
  margin-top: 20px;
  padding: 40px 20px;
  border: 2px solid rgb(99, 116, 198);
  border-radius: 15px;
  box-shadow: rgba(99, 98, 98, 0.35) 0px 5px 15px;
}
.container__choise {
  display: flex;
  justify-content: space-between;
}

.choise {

  max-width: 16ch;
}
.table {
}
.table__items {
  margin: 5px 0px;
  width: max-content;
  min-width: 200px;
  padding: 5px 10px;
  border: 2px solid rgb(99, 116, 198);
  border-radius: 15px;
  box-shadow: rgba(99, 98, 98, 0.35) 0px 5px 15px;
}
.items {
  display: flex;
  gap: 4px;
}
.button__container {
}

.button {
  background: linear-gradient(to bottom right, #ef4765, #ff9a5a);
  border: 0;
  margin: 0px 5px;
  position: relative;
  border-radius: 12px;
  color: #ffffff;
  cursor: pointer;
  display: inline-block;
  font-family: -apple-system, system-ui, "Segoe UI", Roboto, Helvetica, Arial,
    sans-serif;
  font-size: 16px;
  font-weight: 500;
  line-height: 2.5;
  outline: transparent;
  padding: 0 1rem;
  text-align: center;
  text-decoration: none;
  transition: box-shadow 0.2s ease-in-out;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
  transition: transform 0.4s ease-in-out;
}

.button:not([disabled]):focus {
  box-shadow: 0 0 0.25rem rgba(0, 0, 0, 0.5),
    -0.125rem -0.125rem 1rem rgba(239, 71, 101, 0.5),
    0.125rem 0.125rem 1rem rgba(255, 154, 90, 0.5);
}

.button:not([disabled]):hover {
  box-shadow: 0 0 0.25rem rgba(0, 0, 0, 0.5),
    -0.125rem -0.125rem 1rem rgba(239, 71, 101, 0.5),
    0.125rem 0.125rem 1rem rgba(255, 154, 90, 0.5);
}
.blue {
  background: linear-gradient(to bottom right, #476eef, #e95aff);
}
.blue:not([disabled]):focus {
  box-shadow: 0 0 0.25rem rgba(0, 0, 0, 0.5), -0.125rem -0.125rem 1rem #476eef,
    0.125rem 0.125rem 1rem rgba(255, 154, 90, 0.5);
}

.blue:not([disabled]):hover {
  box-shadow: 0 0 0.25rem rgba(0, 0, 0, 0.5), -0.125rem -0.125rem 1rem #476eef,
    0.125rem 0.125rem 1rem rgba(255, 154, 90, 0.5);
}
.popup {
}
.popup__background {
  width: 100%;
  height: 100%;
  position: fixed;
  background: #000;
  opacity: 40%;
  top: 0px;
  right: 0px;
  bottom: 0px;
  left: 0px;
}
.popup__card {
  display: flex;
  justify-content: center;

  
}
.popup__select{ 
  position:absolute;
  top: 50%;
  
  
}

@media screen and (max-width: 769px) {
  .container__form {
    flex-direction: column;
  }
  .navigation_container {
  }
  .form__currency {
    flex-direction: column;
    
  }
  .input {
    width: auto;
    max-width: 16ch;
  }
  .select {
    max-width: 18ch;
  }
  .result {
 font-size: 14px;
  }
  .container__choise {
    flex-direction: column-reverse;
    justify-content: center;
    padding: 15px 10px;
    width: 0;
  }
  .choise {
    margin-top: 10px;
    max-width: 16ch;
  }
  .button {
    margin: 4px;
  }
}

</style>