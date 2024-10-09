<template>
  <div class="lottery-app">
    <!-- Блок переможців -->
    <div class="card">
      <div class="winner-tags gray-border d-flex">
        <span
          v-for="(winner, index) in winners"
          :key="index"
          class="badge blue d-flex align-items-center"
        >
          {{ winner.name }}
          <button
            @click="removeWinner(index)"
            class="btn btn-sm btn-danger ms-2"
          >
            &times;
          </button>
        </span>
        <span class="badge">Winners</span>
      </div>
      <button
        class="btn btn-primary mt-2"
        :disabled="winners.length >= 3 || participants.length === 0"
        @click="selectWinner"
      >
        New winner
      </button>
    </div>

    <!-- Форма реєстрації -->
    <div class="card">
      <h3>REGISTER FORM</h3>
      <p>Please fill in all the fields.</p>
      <form @submit.prevent="registerParticipant" novalidate>
        <div class="form-group mb-3">
          <label>Name</label>
          <input
            v-model="newParticipant.name"
            type="text"
            class="form-control"
            placeholder="Enter user name"
            :class="{ 'is-invalid': nameError }"
            required
          />
          <div class="invalid-feedback">{{ nameError }}</div>
        </div>
        <div class="form-group mb-3">
          <label>Date of Birth</label>
          <input
            v-model="newParticipant.dateOfBirth"
            type="date"
            class="form-control"
            :max="today"
            :class="{ 'is-invalid': dateError }"
            required
          />
          <div class="invalid-feedback">{{ dateError }}</div>
        </div>
        <div class="form-group mb-3">
          <label>Email</label>
          <input
            v-model="newParticipant.email"
            type="email"
            class="form-control"
            placeholder="Enter email"
            :class="{ 'is-invalid': emailError }"
            required
          />
          <div class="invalid-feedback">{{ emailError }}</div>
        </div>
        <div class="form-group mb-3">
          <label>Phone number</label>
          <input
            v-model="newParticipant.phoneNumber"
            type="tel"
            class="form-control"
            placeholder="Enter phone number"
            :class="{ 'is-invalid': phoneError }"
            required
          />
          <div class="invalid-feedback">{{ phoneError }}</div>
        </div>
        <button type="submit" class="btn btn-primary">Save</button>
      </form>
    </div>

    <!-- Таблиця учасників -->
    <div class="card">
      <table class="table table-striped">
        <thead class="table-light">
          <tr>
            <th>#</th>
            <th>Name</th>
            <th>Date of Birth</th>
            <th>Email</th>
            <th>Phone number</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(participant, index) in participants" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ participant.name }}</td>
            <td>{{ participant.dateOfBirth }}</td>
            <td>{{ participant.email }}</td>
            <td>{{ participant.phoneNumber }}</td>
            <td class="d-flex justify-content-center align-items-center">
              <button
                @click="removeParticipant(index)"
                class="btn btn-danger btn-sm delete-btn"
              >
                &times;
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";
import { Validator } from "@/misc/Validator";
import { Participant } from "@/models/Participant";

export default defineComponent({
  name: "LotteryApp",
  setup() {
    const today = new Date().toISOString().split("T")[0];
    const newParticipant = ref<Participant>({
      name: "",
      dateOfBirth: "",
      email: "",
      phoneNumber: "",
    });

    const participants = ref<Participant[]>([]);
    const winners = ref<Participant[]>([]);

    const nameError = ref("");
    const dateError = ref("");
    const emailError = ref("");
    const phoneError = ref("");

    onMounted(() => {
      const storedParticipants = localStorage.getItem("participants");
      if (storedParticipants) {
        participants.value = JSON.parse(storedParticipants);
      }
    });

    const saveParticipantsToLocalStorage = () => {
      localStorage.setItem("participants", JSON.stringify(participants.value));
    };

    const removeParticipant = (index: number) => {
      participants.value.splice(index, 1);
      saveParticipantsToLocalStorage();
    };

    const registerParticipant = () => {
      nameError.value = Validator.validateName(newParticipant.value.name);
      dateError.value = Validator.validateDateOfBirth(
        newParticipant.value.dateOfBirth,
        today
      );
      emailError.value = Validator.validateEmail(newParticipant.value.email);
      phoneError.value = Validator.validatePhoneNumber(
        newParticipant.value.phoneNumber
      );

      if (
        nameError.value ||
        dateError.value ||
        emailError.value ||
        phoneError.value
      ) {
        return;
      }

      participants.value.push({ ...newParticipant.value });
      Object.assign(newParticipant.value, {
        name: "",
        dateOfBirth: "",
        email: "",
        phoneNumber: "",
      });

      saveParticipantsToLocalStorage();
    };

    const selectWinner = () => {
      if (participants.value.length > 0 && winners.value.length < 3) {
        const randomIndex = Math.floor(
          Math.random() * participants.value.length
        );
        const winner = participants.value.splice(randomIndex, 1)[0];
        winners.value.push(winner);
        saveParticipantsToLocalStorage();
      }
    };

    const removeWinner = (index: number) => {
      participants.value.push(winners.value.splice(index, 1)[0]);
      saveParticipantsToLocalStorage();
    };

    return {
      newParticipant,
      participants,
      winners,
      nameError,
      dateError,
      emailError,
      phoneError,
      today,
      registerParticipant,
      selectWinner,
      removeWinner,
      removeParticipant,
    };
  },
});
</script>

<style scoped>
.lottery-app {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
  max-width: 800px;
  margin: 0 auto;
}

.card {
  border: 1px solid #dee2e6;
  border-radius: 8px;
  background-color: #f8f9fa;
  padding: 1.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.winner-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.blue {
  background-color: rgba(13, 110, 253, 0.5);
}

.gray-border {
  border: 1px solid gray;
  border-radius: 10px;
  padding: 10px;
}

.invalid-feedback {
  display: block;
}

.table-light th {
  background-color: #e9ecef;
}

/* .delete-btn {
  margin-bottom: 0px; 
}
 .delete-btn {
  padding-bottom: 0px; 
}

button {
  margin-top: 0px;
} */
.delete-btn {
  padding-bottom: 0px;
}
</style>
