<template>
  <label class="block">
    <span>{{ labelText }}</span>
    <input
      :type="inputType"
      class="mt-1 block w-full"
      :placeholder="placeholder"
      :value="value"
      @input="$emit('update:value', $event.target.value)"
    />
  </label>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from "vue";

const props = defineProps({
  labelText: String,
  inputType: {
    type: String,
    default: "text",
  },
  placeholder: String,
  value: String,
});

const emit = defineEmits(["update:value"]);
</script>

<style scoped>
/* Scoped CSS styles */
</style>
