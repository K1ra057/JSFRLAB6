<template>
  <div v-if="isVisible" class="modal-overlay">
    <div class="modal-content">
      <h3 v-if="mode === 'edit'">Edit Participant</h3>
      <h3 v-else>Confirm Deletion</h3>

      <form v-if="mode === 'edit'" @submit.prevent="updateParticipant">
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

      <div v-else>
        <p v-if="participant">
          Ви дійсно бажаєте видалити учасника "{{ participant.name }}", "{{
            participant.email
          }}"?
        </p>
        <p v-else>Учасник не вибраний.</p>
        <div class="button-group">
          <button class="btn btn-danger" @click="confirm">Так</button>
          <button class="btn btn-secondary" @click="closeModal">Ні</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onBeforeUnmount, onMounted, ref, watch } from "vue";
import { Participant } from "@/models/Participant";
import { Validator } from "@/misc/Validator";

export default defineComponent({
  name: "ModalUnified",
  props: {
    isVisible: {
      type: Boolean,
      required: true,
    },
    participant: {
      type: Object as () => Participant | null,
      required: true,
    },
    mode: {
      type: String,
      required: true,
      validator: (value: string) => ["edit", "confirm"].includes(value),
    },
  },
  emits: ["update:participant", "close", "confirm"],
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

    const confirm = () => {
      emit("confirm");
      closeModal();
    };

    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === "Escape") {
        closeModal();
      } else if (event.key === "Enter" && props.mode === "confirm") {
        confirm();
      } else if (event.key === "Enter" && props.mode === "edit") {
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
      confirm,
      closeModal,
    };
  },
});
</script>

<style scoped>
/* ваші стилі залишаються без змін */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  width: 300px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.button-group {
  display: flex;
  justify-content: space-between;
}

.btn {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}
</style>
