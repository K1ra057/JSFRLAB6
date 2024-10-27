<template>
  <div class="card">
    <SearchBar @filter-by-name="filterParticipants" />
    <table class="table table-striped">
      <thead>
        <tr>
          <th>#</th>
          <th>
            Name
            <button
              @click="sortByName"
              aria-label="Sort by Name"
              class="sort-button"
            >
              <img src="@/assets/sort-alpha-down.svg" alt="AZ" />
            </button>
          </th>
          <th>
            Date of Birth
            <button
              @click="sortByDateOfBirth"
              aria-label="Sort by Date of Birth"
              class="sort-button"
            >
              <img src="@/assets/sort-down.svg" alt="^" />
            </button>
          </th>
          <th>Email</th>
          <th>Phone number</th>
          <th>Edit</th>
          <th>Delete</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(participant, index) in filteredParticipants" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ participant.name }}</td>
          <td>{{ participant.dateOfBirth }}</td>
          <td>{{ participant.email }}</td>
          <td>{{ participant.phoneNumber }}</td>
          <td>
            <button @click="openModal(participant, 'edit')">Edit</button>
          </td>
          <td>
            <button
              class="delete-button"
              @click="openModal(participant, 'confirm')"
            >
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <ModalUnified
      v-if="isModalVisible"
      :isVisible="isModalVisible"
      :participant="selectedParticipant"
      :mode="modalMode"
      @update:participant="updateParticipant"
      @confirm="deleteParticipant"
      @close="closeModal"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watch } from "vue";
import { Participant } from "@/models/Participant";
import ModalUnified from "@/components/ModalUnified.vue";
import SearchBar from "@/components/SearchBar.vue";
import MyStorage from "@/misc/MyStorage";

export default defineComponent({
  name: "ParticipantsTable",
  props: {
    participants: {
      type: Array as () => Participant[],
      required: true,
      default: () => [],
    },
  },
  components: {
    ModalUnified,
    SearchBar,
  },
  setup(props) {
    const isModalVisible = ref(false);
    const selectedParticipant = ref<Participant | null>(null);
    const modalMode = ref<"edit" | "confirm">("edit");
    const localParticipants = ref<Participant[]>([...props.participants]);
    const searchTerm = ref("");
    watch(
      () => props.participants,
      (newParticipants) => {
        localParticipants.value = [...newParticipants];
      },
      { immediate: true } // Викликає спостерігача одразу після створення компонента
    );
    const filteredParticipants = computed(() => {
      return localParticipants.value.filter((participant) =>
        participant.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      );
    });

    const openModal = (participant: Participant, mode: "edit" | "confirm") => {
      selectedParticipant.value = { ...participant };
      modalMode.value = mode;
      isModalVisible.value = true;
    };

    const closeModal = () => {
      isModalVisible.value = false;
      selectedParticipant.value = null;
    };

    const updateParticipant = (updatedParticipant: Participant) => {
      const index = localParticipants.value.findIndex(
        (p) => p.email === updatedParticipant.email
      );
      if (index !== -1) {
        localParticipants.value[index] = updatedParticipant;
        MyStorage.saveParticipants(localParticipants.value);
      }
      closeModal();
    };

    const deleteParticipant = () => {
      if (selectedParticipant.value) {
        const emailToDelete = selectedParticipant.value.email;
        localParticipants.value = localParticipants.value.filter(
          (participant) => participant.email !== emailToDelete
        );
        MyStorage.saveParticipants(localParticipants.value);
      }
      closeModal();
    };

    const sortByName = () => {
      localParticipants.value.sort((a, b) => a.name.localeCompare(b.name));
    };

    const sortByDateOfBirth = () => {
      localParticipants.value.sort((a, b) => {
        return (
          new Date(a.dateOfBirth).getTime() - new Date(b.dateOfBirth).getTime()
        );
      });
    };

    const filterParticipants = (search: string) => {
      searchTerm.value = search;
    };

    return {
      isModalVisible,
      selectedParticipant,
      modalMode,
      filteredParticipants,
      openModal,
      closeModal,
      updateParticipant,
      deleteParticipant,
      sortByName,
      sortByDateOfBirth,
      filterParticipants,
    };
  },
});
</script>

<style scoped>
.sort-button {
  background: none;
  border: none;
  cursor: pointer;
  margin-left: 5px;
}
.sort-button i {
  font-size: 16px;
}
.delete-button {
  background-color: red;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 5px;
  cursor: pointer;
}

.delete-button:hover {
  background-color: darkred;
}
</style>
