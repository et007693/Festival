<template>
  <div class="p-6 max-w-5xl mx-auto">
    <h2 class="text-2xl font-bold mb-4">📅 KCISA 문화데이터 캘린더</h2>
    <FestivalMain :setSelectedEvent="setSelectedEvent" />
    <BaseModal v-model="showModal">
      <div v-if="selectedEvent" class="mt-4 p-4 border rounded shadow">
        <h3 class="text-xl font-semibold mb-2">{{ selectedEvent.title }}</h3>
        <p>
          <strong>기간:</strong> {{ selectedEvent.start }} ~
          {{ selectedEvent.end }}
        </p>
        <p><strong>장소:</strong> {{ selectedEvent.place }}</p>
        <p>
          <strong>링크:</strong>
          <a
            :href="selectedEvent.url"
            target="_blank"
            class="text-blue-600 hover:underline"
            >{{ selectedEvent.url }}</a
          >
        </p>
        <div>
          <strong>설명:</strong>
          <div v-html="selectedEvent.description"></div>
        </div>
      </div>
    </BaseModal>
  </div>
</template>

<script setup>
import { ref } from "vue";
import BaseModal from "./component/BaseModal.vue";
import FestivalMain from "./pages/FestivalMain.vue";

const selectedEvent = ref(null);
const setSelectedEvent = (event) => {
  selectedEvent.value = event;
  showModal.value = true;
};
const showModal = ref(false);
</script>
