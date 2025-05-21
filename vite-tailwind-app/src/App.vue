<template>
  <div class="p-6 max-w-5xl mx-auto">
    <h2 class="text-2xl font-bold mb-4">📅 KCISA 문화데이터 캘린더</h2>
<<<<<<< HEAD
    <!-- <vue-cal
      style="height: 600px"
      :events="events"
      default-view="month"
      :on-event-click="onEventClick"
      :selected-date="new Date('2020-10-01')"
    /> -->
=======
>>>>>>> refs/remotes/origin/master

    <FestivalMain :setSelectedEvent="setSelectedEvent" />

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
        >
          {{ selectedEvent.url }}
        </a>
      </p>
      <div>
        <strong>설명:</strong>
        <div v-html="selectedEvent.description"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import FestivalMain from "./pages/FestivalMain.vue";

const selectedEvent = ref(null);
<<<<<<< HEAD

const calendarOptions = ref({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
<<<<<<< HEAD
  initialDate: "2020-10-01",
  // dateClick: (arg) => alert("date click! " + arg.dateStr),
=======
  height: "auto",
>>>>>>> refs/remotes/origin/master
  events: [],
  eventClick: function (info) {
    selectedEvent.value = {
      title: info.event.title,
      start: info.event.startStr,
      end: info.event.endStr,
      place: info.event.extendedProps.place || "",
<<<<<<< HEAD
      url: info.event.url,
=======
      url: info.event.extendedProps.url,
>>>>>>> refs/remotes/origin/master
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
<<<<<<< HEAD

    const fullCalendarEvents = items
      .map((item) => {
        const { start, end } = parsePeriod(item.PERIOD);

        if (!start || !end) return null;
        return {
          title: item.TITLE,
          start,
          end,
          extendedProps: {
            place: item.sourceTitle || "",
            url: item.URL,
            description: item.description,
          },
        };
      })
      .filter(Boolean);

    calendarOptions.value.events = fullCalendarEvents;
=======
>>>>>>> refs/remotes/origin/master

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

<<<<<<< HEAD
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
=======
<style scoped></style>
>>>>>>> refs/remotes/origin/master
=======
const setSelectedEvent = (event) => {
  selectedEvent.value = event;
};
</script>
>>>>>>> b12efec14445049d8b018eff2598cbdbe4c94dd5
