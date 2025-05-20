<template>
  <div class="p-6 max-w-5xl mx-auto">
    <h2 class="text-2xl font-bold mb-4">📅 KCISA 문화데이터 캘린더</h2>
    <vue-cal
      style="height: 600px"
      :events="events"
      default-view="month"
      :on-event-click="onEventClick"
      :selected-date="new Date('2020-10-01')"
    />

    <div v-if="selectedEvent" class="mt-4 p-4 border rounded shadow">
      <h3 class="text-xl font-semibold mb-2">{{ selectedEvent.title }}</h3>
      <p>
        <strong>기간:</strong> {{ selectedEvent.start }} ~
        {{ selectedEvent.end }}
      </p>
      <p><strong>장소:</strong> {{ selectedEvent.place }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import VueCal from "vue-cal";
import "vue-cal/dist/vuecal.css";
import axios from "axios";
import { XMLParser } from "fast-xml-parser";

const events = ref([]);
const selectedEvent = ref(null);

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
      "http://api.kcisa.kr/API_CNV_050/request?serviceKey=25a8f85e-7a1f-4849-b9b9-c1fdae8e9e92&numOfRows=100",
      { responseType: "text" }
    );
    console.log(res.data); // XML -> JSON 결과
    const json = JSON.parse(res.data);
    const items = json.response.body.items.item;

    events.value = items
      .map((item) => {
        console.log(item); // XML -> JSON 결과
        const { start, end } = parsePeriod(item.period);

        if (!start || !end) return null;
        return {
          title: item.title,
          start,
          end,
          place: item.sourceTitle || "",
          url: item.url,
          description: item.description,
        };
      })
      .filter(Boolean); // null 제거
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

<style scoped>
.vuecal {
  font-size: 0.9rem;
}
</style>
