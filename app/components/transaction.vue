<template>
  <div class="grid grid-cols-2 py-4 border-b">
    <div class="flex items-center justify-between">
      <div class="flex items-center space-x-1">
        <UIcon :name="icon" :class="[iconColor]" />
        <div>{{ type }}</div>
      </div>
      <UBadge color="white">{{ category }}</UBadge>
    </div>
    <div class="flex items-center justify-end space-x-2">
      <div>${{ amount }}</div>
      <div>
        <UDropdownMenu :items="items">
          <UButton
            color="white"
            variant="ghost"
            trailing-icon="i-heroicons-ellipsis-horizontal"
          />
        </UDropdownMenu>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  id: String,
  amount: Number,
  type: String,
  category: String,
});
const emit = defineEmits(['deleted']);
const icon = computed(() => {
  return props.type === "income"
    ? "i-heroicons-arrow-up-right"
    : "i-heroicons-arrow-down-left";
});
const iconColor = computed(() =>
  props.type === "income" ? "text-green-600" : "text-red-600"
);
const isLoading = ref(false);
const toast = useToast();
const supabase = useSupabaseClient();

const deleteTransaction = async () => {
  isLoading.value = true;
  const { error } = await supabase
    .from('transactions')
    .delete()
    .eq('id', props.id);
  
  if (error) {
    toast.add({
      title: "Error",
      description: error.message,
      color: "error"
    });
  } else {
    toast.add({
      title: "Success",
      description: "Transaction deleted successfully",
      color: "success"
    });
    emit('deleted',props.id);

  }
  isLoading.value = false;
};

const editTransaction = async () => {
  console.log("edit");
};
const items = [
  [
    {
      label: "Edit",
      icon: "i-heroicons-pencil-square-20-solid",
      onSelect: editTransaction,
    },
    {
      label: "Delete",
      icon: "i-heroicons-trash-20-solid",
      onSelect: deleteTransaction,
    },
  ],
];
</script>

<style lang="scss" scoped></style>
