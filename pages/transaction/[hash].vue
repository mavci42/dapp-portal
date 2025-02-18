<template>
  <div v-if="transaction">
    <DepositSubmitted v-if="transaction.type === 'deposit'" :transaction="transaction" />
    <TransferSubmitted v-else-if="transaction.type === 'transfer'" :transaction="transaction" />
    <WithdrawalSubmitted v-else-if="transaction.type === 'withdrawal'" :transaction="transaction" />
  </div>
  <div v-else>
    <h1 class="h1 mb-8 mt-4 text-center">Transaction not found</h1>
    <p class="mb-block-gap text-center">
      The transaction you are looking for is not found. It is possible you submitted it from another browser or device.
    </p>
    <CommonButton as="RouterLink" :to="{ name: 'assets' }" class="mt-block-gap" variant="primary">
      Go to Assets page
    </CommonButton>
  </div>
</template>

<script lang="ts" setup>
import DepositSubmitted from "@/views/transactions/DepositSubmitted.vue";
import TransferSubmitted from "@/views/transactions/TransferSubmitted.vue";
import WithdrawalSubmitted from "@/views/transactions/WithdrawalSubmitted.vue";

const route = useRoute();

const transactionStatusStore = useZkSyncTransactionStatusStore();

const completedTransaction = ref<TransactionInfo | null>(null);
const savedTransaction = computed(() => {
  if (typeof route.params.hash !== "string") return;
  return transactionStatusStore.getTransaction(route.params.hash);
});
const transaction = computed(() => {
  return completedTransaction.value || savedTransaction.value;
});

watch(
  () => route.params.hash,
  () => {
    completedTransaction.value = null;
    if (!savedTransaction.value) return;

    transactionStatusStore.waitForCompletion(savedTransaction.value).then((_completedTransaction) => {
      if (route.params.hash === _completedTransaction.transactionHash) {
        completedTransaction.value = _completedTransaction;
      }
    });
  },
  { immediate: true }
);
</script>
