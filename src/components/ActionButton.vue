<template>
  <button
    :type="type"
    class="btn btn-primary"
    :disabled="disabled"
    @click="$emit('click')"
  >
    {{ text }}
  </button>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "ActionButton",
  props: {
    text: {
      type: String,
      required: true,
    },
    type: {
      type: String as () => "button" | "submit" | "reset",
      default: "button",
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  },
});
</script>

<style scoped></style>
