<template>
  <section class="flex items-center justify-between mb-10">
    <h1 class="text-4xl font-extrabold">Summary</h1>
  </section>
  <section
    class="grid grid-col-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10"
  >
    <Trend
      color="green"
      title="income"
      :amount="incomeTotal"
      :last-amount="3000"
      :loading="isLoading"
    />
    <Trend
      color="red"
      title="Expense"
      :amount="expenseTotal"
      :last-amount="10000"
      :loading="isLoading"
    />
    <Trend
      color="green"
      title="Investments"
      :amount="6000"
      :last-amount="7000"
      :loading="isLoading"
    />
    <Trend
      color="red"
      title="Saving"
      :amount="8000"
      :last-amount="6000"
      :loading="isLoading"
    />
  </section>
  <section class="flex justify-between mb-10">
    <div>
      <h2 class="text-2xl font-extrabold">Transactions</h2>
      <div class="text-gray-500 dark:text-gray-400">
        You have {{ incomeCount }} incomes and {{ expenseCount }} expenses this
        period
      </div>
    </div>
    <div>
      <UButton
        icon="i-heroicons-plus-circle"
        color="neutral"
        variant="solid"
        label="Add"
        @click="isOpen = true"
      />
      <TransactionModal v-model="isOpen" />
    </div>
  </section>
  <section>
    <div
      v-for="(transactionOnDay, date) in transactionsGroupedByDate"
      :key="date"
    >
      <DailyTransactionSummary :date="date" :transactions="transactionOnDay" />
      <Transaction
        v-for="transaction in transactionOnDay"
        :id="transaction.id"
        :amount="transaction.amount"
        :type="transaction.type"
        :category="transaction.category"
        :key="transaction.id"
        @deleted="refreshTransactions"
      />
    </div>
  </section>
</template>
<script setup lang="ts">
interface Transaction {
  id: string;
  amount: number;
  type: string;
  category: string;
  created_at?: string;
}

const transactions = ref<Transaction[]>([]);
const supabase = useSupabaseClient();
const isLoading = ref(false);
const isOpen = ref(false);

const income = computed(() =>
  transactions.value.filter((t) => t.type === "Income")
);
const expense = computed(() =>
  transactions.value.filter((t) => t.type === "Expense")
);
const incomeCount = computed(() => income.value.length);
const expenseCount = computed(() => expense.value.length);

const incomeTotal = computed(() =>
  income.value.reduce((sum, transaction) => sum + transaction.amount, 0)
);
const expenseTotal = computed(() =>
  expense.value.reduce((sum, transaction) => sum + transaction.amount, 0)
);

const { data, pending, error, refresh } = await useAsyncData<Transaction[]>(
  "transactions",
  async () => {
    const { data, error } = await supabase.from("transactions").select();
    if (error) {
      console.error("Error loading transactions:", error);
    }
    return data || [];
  }
);

if (error.value) {
  console.error("Error loading transactions:", error.value);
}

transactions.value = data.value || [];

const refreshTransactions = async () => {
  await refresh();
  transactions.value = data.value || [];
};
const transactionsGroupedByDate = computed(() => {
  let grouped: Record<string, Transaction[]> = {};
  for (const transaction of transactions.value) {
    if (!transaction.created_at) continue;
    const date = new Date(transaction.created_at).toISOString().split("T")[0];
    if (date) {
      if (!grouped[date]) {
        grouped[date] = [];
      }
      grouped[date].push(transaction);
    }
  }
  return grouped;
});
</script>
