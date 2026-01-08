<template>
  <UModal v-model:open="isOpen">
    <template #header>
      <div class="w-full">
        <h3 class="text-base font-semibold leading-6">Add Transaction</h3>
      </div>
    </template>
    <template #body>
      <UForm :state="state" :schema="schema" ref="form" @submit="save">
        <UFormField label="type" :required="true" name="type" class="mb-4">
          <USelect
            placeholder="Select the Transaction type"
            :items="types"
            class="w-full"
            v-model="state.type"
          />
        </UFormField>
        <UFormField label="Amount" :required="true" name="amount" class="mb-4">
          <UInput
            type="number"
            placeholder="Amount"
            class="w-full"
            v-model.number="state.amount"
          />
        </UFormField>
        <UFormField
          label="Transaction date"
          :required="true"
          name="created_at"
          class="mb-4"
        >
          <UInput
            type="date"
            icon="i-heroicons-calendar-days-20-solid"
            class="w-full"
            v-model="state.created_at"
          />
        </UFormField>
        <UFormField
          label="Description"
          hint="optional"
          name="description"
          class="mb-4"
        >
          <UInput
            placeholder="Description"
            class="w-full"
            v-model="state.description"
          />
        </UFormField>
        <UFormField
          label="category"
          :required="true"
          name="category"
          class="mb-4"
          v-if="state.type === 'Expense'"
        >
          <USelect
            placeholder="Category"
            :items="categories"
            class="w-full"
            v-model="state.category"
          />
        </UFormField>
        <UButton
          type="submit"
          variant="solid"
          label="save"
          :loading="isLoading"
        />
      </UForm>
    </template>
  </UModal>
</template>

<script setup>
import { z } from "zod";
const categories = ref(["Food", "Housing", "Car", "Entertainment"]);
const types = ref(["Income", "Expense", "Saving", "Investment"]);
const props = defineProps({
  modelValue: Boolean,
});
const emit = defineEmits(["update:modelValue", "saved"]);
const defaultSchema = z.object({
  created_at: z.string("date is required"),
  description: z.string().optional(),
  amount: z.number().positive("amount must be more than 0"),
});
const incomeSchema = z.object({
  type: z.literal("Income"),
});
const expenseSchema = z.object({
  type: z.literal("Expense"),
  category: z.enum(categories),
});
const investmentSchema = z.object({
  type: z.literal("Investment"),
});
const savingSchema = z.object({
  type: z.literal("Saving"),
});
const schema = z.intersection(
  z.discriminatedUnion("type", [
    incomeSchema,
    expenseSchema,
    investmentSchema,
    savingSchema,
  ]),
  defaultSchema
);

const form = ref();
const supabase = useSupabaseClient();
const toast = useToast();
const isLoading = ref(false);
const save = async () => {
  form.value.validate();
  isLoading.value = true;
  try {
    const { error } = await supabase
      .from("transactions")
      .upsert({ ...state.value });
    if (!error) {
      toast.add({
        title: "Success",
        description: "Transaction added successfully",
        color: "success",
      });
      isOpen.value = false;
      emit("saved");
    }
  } catch (e) {
    toast.add({
      title: "Transaction not saved",
      description: e.message,
      color: "error",
    });
  } finally {
    isLoading.value = false;
  }
};
const initialState = {
  type: undefined,
  amount: 0,
  created_at: undefined,
  description: undefined,
  category: undefined,
};
const state = ref({
  ...initialState,
});
const resetForm = () => {
  Object.assign(state.value, initialState);
  form.value.clear();
};

const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => {
    if (!value) resetForm();
    emit("update:modelValue", value);
  },
});
</script>
