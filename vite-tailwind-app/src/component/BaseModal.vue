<template>
  <div
    v-if="modelValue"
    class="fixed inset-0 bg-black/40 flex items-center justify-center z-50"
    @click.self="$emit('update:modelValue', false)"
  >
    <div
      class="relative bg-white rounded-lg shadow-lg w-[90%] max-h-[80vh] overflow-y-auto max-w-screen-md p-6"
    >
      <div class="flex justify-end gap-3">
        <button @click="$emit('update:modelValue', false)" aria-label="닫기">
          &times;
        </button>
      </div>
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
defineProps({ modelValue: Boolean });
defineEmits(["update:modelValue"]);
</script>
