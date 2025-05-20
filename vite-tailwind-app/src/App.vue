<template>
  <div class="p-6 max-w-5xl mx-auto">
    <h2 class="text-2xl font-bold mb-4">📅 KCISA 문화데이터 캘린더</h2>

    <FullCalendar :options="calendarOptions" />

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
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import axios from "axios";
import { XMLParser } from "fast-xml-parser";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";

const events = ref([]);
const selectedEvent = ref(null);

const calendarOptions = ref({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  height: "auto",
  events: [],
  eventClick: function (info) {
    selectedEvent.value = {
      title: info.event.title,
      start: info.event.startStr,
      end: info.event.endStr,
      place: info.event.extendedProps.place || "",
      url: info.event.extendedProps.url,
      description: info.event.extendedProps.description,
    };
  },
});

function formatDate(yyyymmdd) {
  return `${yyyymmdd.slice(0, 4)}-${yyyymmdd.slice(4, 6)}-${yyyymmdd.slice(
    6,
    8
  )}`;
}

function onEventClick(event) {
  selectedEvent.value = event;
}

async function fetchKcisaData() {
  const parser = new XMLParser();
  try {
    const res = await axios.get(
      "http://api.kcisa.kr/openapi/API_CCA_145/request",
      {
        params: {
          serviceKey: "26fad05b-3663-4a81-82ca-2df2ada80ae9",
          numOfRows: 10,
          pageNo: 1,
        },
        headers: {
          Accept: "application/json",
        },
      }
    );
    const items = res.data.response.body.items.item;

    const uniqueEventsMap = new Map();

    items.forEach((item) => {
      const { start, end } = parsePeriod(item.PERIOD);
      if (!start || !end) return;

      const key = item.TITLE;
      if (!uniqueEventsMap.has(key)) {
        uniqueEventsMap.set(key, {
          title: item.TITLE,
          start,
          end,
          extendedProps: {
            place: item.CNTC_INSTT_NM || "",
            url: item.URL,
            description: item.DESCRIPTION,
          },
        });
      }
    });

    calendarOptions.value.events = Array.from(uniqueEventsMap.values());
  } catch (err) {
    console.error("데이터 로딩 실패:", err);
  }
}

function parsePeriod(period) {
  if (!period || !period.includes("~")) return { start: null, end: null };

  const [start, end] = period.split("~").map((s) => s.trim());
  return {
    start: start || null,
    end: end || start || null, // 종료일 없으면 시작일로 채움
  };
}

onMounted(fetchKcisaData);
</script>

<style scoped></style>
