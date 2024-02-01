<template>
  <div class="editable-div" contenteditable="true" @input="updateValue" placeholder="Type your text here">{{ content }}
  </div>
</template>
  
<script setup>
import { ref, watch, defineEmits } from "vue";

const props = defineProps({
  modelValue: String,
});

const emit = defineEmits(["update:modelValue"]);

const content = ref(props.modelValue);

watch(() => props.modelValue, (newValue) => {
  content.value = newValue; // Update the internal state when the prop changes
});

watch(content, (newValue) => {
  emit("update:modelValue", newValue);
});

const updateValue = (event) => {
  content.value = event.target.innerText;
};

</script>
  