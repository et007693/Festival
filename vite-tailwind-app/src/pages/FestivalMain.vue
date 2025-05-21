<template>
  <div class="p-4">
    <!-- 검색 입력창 및 버튼 -->
    <div class="mb-4 flex gap-2">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="행사를 검색하세요"
        @keyup.enter="onSearchClick"
        class="p-2 border rounded w-full"
      />
      <button
        @click="onSearchClick"
        class="px-4 py-2 bg-blue-500 text-white rounded w-[10%]"
      >
        검색
      </button>
    </div>

    <FullCalendar
      :options="calendarOptions"
      ref="calendarRef"
      @datesSet="onDatesSet"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import multimonthPlugin from "@fullcalendar/multimonth";
import axios from "axios";
import { XMLParser } from "fast-xml-parser";

// Props
const props = defineProps({
  setSelectedEvent: Function,
});

const searchQuery = ref(""); // 검색어
const allEvents = ref([]); // 전체 이벤트 저장
const calendarRef = ref(null);

const calendarOptions = ref({
  plugins: [multimonthPlugin, dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  height: "auto",
  events: [],
  headerToolbar: {
    left: "prev,next today",
    center: "title",
    right: "multiMonthYear,dayGridMonth,dayGridWeek",
  },
  eventDidMount(info) {
    info.el.style.marginBottom = "4px";
    info.el.style.padding = "1px 10px";
  },
  eventClick(info) {
    props.setSelectedEvent({
      title: info.event.title,
      start: info.event.startStr,
      end: info.event.endStr,
      place: info.event.extendedProps.place || "",
      url: info.event.extendedProps.url,
      description: info.event.extendedProps.description,
      image: info.event.extendedProps.image || "",
    });
  },
});

// 검색 버튼 클릭 or 엔터 시 호출
function onSearchClick() {
  const query = searchQuery.value.trim().toLowerCase();
  if (!query) {
    calendarOptions.value.events = [...allEvents.value]; // 검색어 없으면 전체
    return;
  }

  calendarOptions.value.events = allEvents.value.filter((event) =>
    event.title.toLowerCase().includes(query)
  );
}

// API 데이터 가져오기
async function fetchKcisaData() {
  const parser = new XMLParser();
  try {
    const res = await axios.get(
      "http://api.kcisa.kr/openapi/API_CCA_145/request",
      {
        params: {
          serviceKey: "26fad05b-3663-4a81-82ca-2df2ada80ae9",
          numOfRows: 100,
          pageNo: 1,
        },
        headers: { Accept: "application/json" },
      }
    );

    const items = res.data.response.body.items.item || [];
    const uniqueEventsMap = new Map();

    items.forEach((item) => {
      const { start, end } = parsePeriod(item.PERIOD);
      if (!start || !end) return;

      const key = item.TITLE;
      if (!uniqueEventsMap.has(key)) {
        const eventColor = getColorByCategory(item.GENRE);
        uniqueEventsMap.set(key, {
          title: item.TITLE,
          start,
          end,
          backgroundColor: hexToRgba(eventColor[0], 0.3),
          borderColor: hexToRgba(eventColor[0], 0.7),
          textColor: eventColor[1],
          extendedProps: {
            place: item.CNTC_INSTT_NM || "",
            url: item.URL,
            description: item.DESCRIPTION,
            image: item.IMAGE_OBJECT,
          },
        });
      }
    });

    const events = Array.from(uniqueEventsMap.values());
    allEvents.value = events;
    calendarOptions.value.events = events;
  } catch (err) {
    console.error("데이터 로딩 실패:", err);
  }
}

function parsePeriod(period) {
  if (!period || !period.includes("~")) return { start: null, end: null };
  const [start, end] = period.split("~").map((s) => s.trim());
  return { start, end: end || start };
}

function hexToRgba(hex, alpha = 0.4) {
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  return `rgba(${r}, ${g}, ${b}, ${alpha})`;
}

function getColorByCategory(category) {
  switch (category) {
    case "예정전시":
      return ["#34d399", "#065f46"];
    case "전시":
      return ["#60a5fa", "#1e3a8a"];
    case "현재전시":
      return ["#fbbf24", "#78350f"];
    default:
      return ["#a78bfa", "#4c1d95"];
  }
}

onMounted(fetchKcisaData);
</script>
