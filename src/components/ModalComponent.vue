<template>
  <div v-if="isVisible" class="modal-overlay">
    <div class="modal-content">
      <h3>Edit Participant</h3>
      <form @submit.prevent="updateParticipant">
        <div class="form-group">
          <label>Name:</label>
          <input v-model="localParticipant.name" type="text" required />
          <span v-if="nameError" class="text-danger">{{ nameError }}</span>
        </div>
        <div class="form-group">
          <label>Date of Birth:</label>
          <input v-model="localParticipant.dateOfBirth" type="date" required />
          <span v-if="dateError" class="text-danger">{{ dateError }}</span>
        </div>
        <div class="form-group">
          <label>Email:</label>
          <input v-model="localParticipant.email" type="email" required />
          <span v-if="emailError" class="text-danger">{{ emailError }}</span>
        </div>
        <div class="form-group">
          <label>Phone Number:</label>
          <input v-model="localParticipant.phoneNumber" type="tel" required />
          <span v-if="phoneError" class="text-danger">{{ phoneError }}</span>
        </div>
        <div class="button-group">
          <button type="submit" class="btn btn-primary">Update Data</button>
          <button type="button" @click="closeModal" class="btn btn-secondary">
            Cancel
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onBeforeUnmount, onMounted, ref, watch } from "vue";
import { Participant } from "@/models/Participant";
import { Validator } from "@/misc/Validator";

export default defineComponent({
  name: "ModalComponent",
  props: {
    isVisible: {
      type: Boolean,
      required: true,
    },
    participant: {
      type: Object as () => Participant | null,
      required: true,
    },
  },
  emits: ["update:participant", "close"],
  setup(props, { emit }) {
    const localParticipant = ref<Participant>({
      name: "",
      dateOfBirth: "",
      email: "",
      phoneNumber: "",
      ...props.participant,
    });

    const nameError = ref("");
    const dateError = ref("");
    const emailError = ref("");
    const phoneError = ref("");

    watch(
      () => props.participant,
      (newParticipant) => {
        if (newParticipant) {
          localParticipant.value = { ...newParticipant };
        }
      }
    );

    const validateFields = () => {
      nameError.value = Validator.validateName(localParticipant.value.name);
      dateError.value = Validator.validateDateOfBirth(
        localParticipant.value.dateOfBirth,
        new Date().toISOString().split("T")[0]
      );
      emailError.value = Validator.validateEmail(
        localParticipant.value.email,
        true
      );
      phoneError.value = Validator.validatePhoneNumber(
        localParticipant.value.phoneNumber
      );
    };

    const updateParticipant = () => {
      validateFields();
      if (
        !nameError.value &&
        !dateError.value &&
        !emailError.value &&
        !phoneError.value
      ) {
        emit("update:participant", localParticipant.value);
        closeModal();
      }
    };

    const closeModal = () => {
      emit("close");
    };

    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === "Escape") {
        closeModal();
      } else if (event.key === "Enter") {
        updateParticipant();
      }
    };

    onMounted(() => {
      window.addEventListener("keydown", handleKeyDown);
    });

    onBeforeUnmount(() => {
      window.removeEventListener("keydown", handleKeyDown);
    });

    return {
      localParticipant,
      nameError,
      dateError,
      emailError,
      phoneError,
      updateParticipant,
      closeModal,
    };
  },
});
</script>

<style scoped>
/* ваші стилі залишаються без змін */
</style>
