<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
    <IncomeExpense :income="income" :expenses="expenses" />
    <TransactionList
      @transactionDeleted="handleTransactionDeleted"
      :transactions="transactions"
    />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>
<script setup>
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpense from "./components/IncomeExpense.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";

import { useToast } from "vue-toastification";
const toast = useToast();

import { ref, computed, onMounted } from "vue";

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem("transactions"));
  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Get expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Add transaction
const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount,
  });
  toast.success("Transaction Success");

  saveTransactionsToLocalstrage(transactions.value);
};

const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id
  );
  saveTransactionsToLocalstrage(transactions.value);
  toast.success("Transaction deleted");
};

// Save to localstorage
const saveTransactionsToLocalstrage = (transactionsData) => {
  localStorage.setItem("transactions", JSON.stringify(transactionsData));
};

const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000);
};
</script>