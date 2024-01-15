<template>
  <label class="flex flex-col">
    <span class="mb-auto">{{ labelText }}</span>
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
input[type="color"] {
  /* -webkit-appearance: none; */
  /* border: none; */
  width: 32px;
  height: 32px;
  border-radius: 9999px;
}
input[type="color"]::-webkit-color-swatch-wrapper {
  padding: 0;
}
input[type="color"]::-webkit-color-swatch {
  /* border: none; */
  border-radius: 9999px;
}
</style>
