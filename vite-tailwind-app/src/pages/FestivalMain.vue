<template>
  <div>
    <div class="p-4">
      <FullCalendar :options="calendarOptions" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import axios from "axios";
import { XMLParser } from "fast-xml-parser";

// props
const props = defineProps({
  setSelectedEvent: Function,
});

const calendarOptions = ref({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  height: "auto",
  events: [],
  eventClick: function (info) {
    props.setSelectedEvent({
      title: info.event.title,
      start: info.event.startStr,
      end: info.event.endStr,
      place: info.event.extendedProps.place || "",
      url: info.event.extendedProps.url,
      description: info.event.extendedProps.description,
    });
  },
});

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
        headers: { Accept: "application/json" },
      }
    );
    const items = res.data.response.body.items.item;

    const uniqueEventsMap = new Map();
    items.forEach((item) => {
      const { start, end } = parsePeriod(item.PERIOD);
      if (!start || !end) return;

      const key = item.TITLE;
      if (!uniqueEventsMap.has(key)) {
        const eventColor = getColorByCategory(
          item.CATEGORY || item.CNTC_INSTT_NM
        );

        uniqueEventsMap.set(key, {
          title: item.TITLE,
          start,
          end,
          color: eventColor, //색상 지정
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
  return { start, end: end || start };
}

// 색상 지정 함수
function getColorByCategory(category) {
  switch (category) {
    case "문화":
      return "#34d399"; // green-400
    case "전시":
      return "#60a5fa"; // blue-400
    case "교육":
      return "#fbbf24"; // yellow-400
    default:
      return "#a78bfa"; // purple-400
  }
}

onMounted(fetchKcisaData);
</script>
