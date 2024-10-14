<template>
  <div class="modal">
    <div class="modal-content">
      <h4>{{ title }}</h4>
      <p>{{ message }}</p>
      <div class="button-container">
        <button @click="$emit('confirm')" class="btn btn-danger">
          {{ confirmText }}
        </button>
        <button @click="$emit('cancel')" class="btn btn-secondary">
          {{ cancelText }}
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    title: {
      type: String,
      required: true,
    },
    message: {
      type: String,
      required: true,
    },
    confirmText: {
      type: String,
      default: "Delete",
    },
    cancelText: {
      type: String,
      default: "Cancel",
    },
  },
});
</script>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  width: 300px; /* Додайте ширину модального вікна */
  text-align: center; /* Центруємо текст */
}

.modal-content button {
  width: 120px; /* Фіксована ширина кнопок */
  margin: 0 5px; /* Невеликий відступ між кнопками */
}

.modal-content .button-container {
  display: flex;
  justify-content: center; /* Центруємо кнопки */
  margin-top: 20px; /* Відступ зверху для кнопок */
}
</style>
