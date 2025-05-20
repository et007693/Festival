<template>
  <div class="p-6 max-w-5xl mx-auto">
    <h2 class="text-2xl font-bold mb-4">📅 KCISA 문화데이터 캘린더</h2>
    <!-- <vue-cal
      style="height: 600px"
      :events="events"
      default-view="month"
      :on-event-click="onEventClick"
      :selected-date="new Date('2020-10-01')"
    /> -->

    <FullCalendar :options="calendarOptions" />

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
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";

const events = ref([]);
const selectedEvent = ref(null);

const calendarOptions = ref({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  initialDate: "2020-10-01",
  // dateClick: (arg) => alert("date click! " + arg.dateStr),
  events: [],
  eventClick: function (info) {
    selectedEvent.value = {
      title: info.event.title,
      start: info.event.startStr,
      end: info.event.endStr,
      place: info.event.extendedProps.place || "",
      url: info.event.url,
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
      "http://api.kcisa.kr/API_CNV_050/request?serviceKey=25a8f85e-7a1f-4849-b9b9-c1fdae8e9e92&numOfRows=100",
      { responseType: "text" }
    );
    console.log(res.data); // XML -> JSON 결과
    const json = JSON.parse(res.data);
    const items = json.response.body.items.item;

    const fullCalendarEvents = items
      .map((item) => {
        const { start, end } = parsePeriod(item.period);

        if (!start || !end) return null;
        return {
          title: item.title,
          start,
          end,
          extendedProps: {
            place: item.sourceTitle || "",
            url: item.url,
            description: item.description,
          },
        };
      })
      .filter(Boolean);

    calendarOptions.value.events = fullCalendarEvents;

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

<script>
import FullCalendar from "@fullcalendar/vue3";

export default {
  components: {
    FullCalendar, // make the <FullCalendar> tag available
  },
  methods: {
    handleDateClick: function (arg) {
      // alert("date click! " + arg.dateStr);
    },
  },
};
</script>

<style scoped>
.vuecal {
  --vuecal-primary: #3b82f6;
  --vuecal-accent: #60a5fa;
  --vuecal-bg: #f9fafb;
  --vuecal-text: #1f2937;
  border-radius: 1rem;
  font-size: 0.95rem;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
