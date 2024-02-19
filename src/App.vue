<template>
  <Header />
  <div class="container">
    <Balance :total="+total"/>
    <IncomeExpenses :income="+income" :expense="+expense"/>
    <TransactionList :transactions="transactions" @transactionDeleted="deleteTransaction"/>
    <AddTransaction @transactionSubmitted="addTransaction"/>
  </div>
</template>

<script setup>
 import Header from './components/Header.vue'
 import Balance from './components/Balance.vue'
 import IncomeExpenses from './components/IncomeExpenses.vue'
 import TransactionList from './components/TransactionList.vue'
 import AddTransaction from './components/AddTransaction.vue'

 import { useToast } from 'vue-toastification';

 import { ref, computed, onMounted } from 'vue'

 const toast = useToast();

 const transactions = ref([]);

 onMounted(() => {
  const storedTransactions = Json.parse(localStorage.getItem('transactions'));

  if (storedTransactions) {
    transactions.value = storedTransactions;
  }
 });

  // Get total
  const total = computed(() => {
    return transactions.value.reduce((acc, transactions) => {
      return acc + transactions.amount;
    }, 0);
  });

  // Get income
  const income = computed(() => {
    return transactions.value
      .filter(transaction => transaction.amount > 0)
      .reduce((acc, transaction) => {
        return acc + transaction.amount;
      }, 0)
      .toFixed(2);
  });

  // Get expense
  const expense = computed(() => {
    return transactions.value
      .filter(transaction => transaction.amount < 0)
      .reduce((acc, transaction) => {
        return acc + transaction.amount;
      }, 0);
  });

  // Add transaction
  const addTransaction = (transactionData) => {
    transactions.value.push({
      id: generateUniqueId(),
      text: transactionData.text,
      amount: +transactionData.amount
    });

    saveToLocalStorage();

    toast.success('Transaction added successfully!');
  };

  // Generate unique ID
  const generateUniqueId = () => {
    return Math.floor(Math.random() * 1000000000);
  };

  // Delete transaction
  const deleteTransaction = (id) => {
    transactions.value = transactions.value.filter(transaction => transaction.id !== id);

    saveToLocalStorage();

    toast.success('Transaction deleted successfully!');
  };

  // Local storage
  const saveToLocalStorage = () => {
    localStorage.setItem('transactions', JSON.stringify(transactions.value));
  };
</script>