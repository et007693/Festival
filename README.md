### **조원: 채상윤, 김세민, 김대환, 송창용**

# 주제: 문화행사정보 캘린더

---

## 프로젝트 배경 및 문제점

- 네이버, KEOA 한국전시주최자협회 등의 **웹 사이트에서 국내 문화 행사 및 전시 등의 정보를 제공**하고 있음.
- 제공하고 있는 정보들은 **민간 단체 행사 및 정부 소속 기관인 문화체육관광부에서 주관하는 전시 및 행사를 포함**하고 있음.
- 민간 단체의 전시 관람 비용은 정부 기관에서 주관하는 전시보다 **높은 가격대가 형성되어 있어 문화 행사 참여에 어려움**이 있음.
- **전시 정보를 한 눈에 확인하는데 어려움**이 있음

![스크린샷 2025-05-21 오전 9.40.07.png](attachment:80c90efd-e49a-4640-8fd4-204a72f3582b:스크린샷_2025-05-21_오전_9.40.07.png)

![스크린샷 2025-05-21 오전 9.42.28.png](attachment:9096a2d5-3036-49c6-80c2-e41e3afb9c36:스크린샷_2025-05-21_오전_9.42.28.png)

[전시회 : 네이버 검색](https://search.naver.com/search.naver?where=nexearch&sm=tab_etc&mra=bjBC&qvt=0&query=%EC%A0%84%EC%8B%9C%ED%9A%8C)

![스크린샷 2025-05-21 오전 9.46.00.png](attachment:9583e428-f156-43ac-b624-71a433f708a2:스크린샷_2025-05-21_오전_9.46.00.png)

![스크린샷 2025-05-21 오전 9.46.10.png](attachment:0b7bb569-6f46-4eb0-870a-1c358d6fe5f9:스크린샷_2025-05-21_오전_9.46.10.png)

[www.keoa.org](http://www.keoa.org/directory/schedule)

## 프로젝트 목표

- 문화관광체육부 소속 기관의 전시 정보 데이터([API](https://www.culture.go.kr/data/openapi/openapiView.do?id=598))를 활용함.
    - 소속 기관: 미술관, 박물관, 문화 정보원 등
- 달력 형태로 정보를 제공하는 웹 어플리케이션 제작.
    - [FullCalendar](https://fullcalendar.io/) 활용

---

# 기능 명세

- [x]  JSON 기반의 API활용해서 정보 가져와 보여 주기(axios)
- [x]  1페이지 기준으로 작성
- [x]  하위컴포넌트를 가져와 보여 주는 방식(props, 필요시 emit)
- [x]  최대한 onMounted, computed, ref, reactive, watch 활용해서 구현
- [x]  UI는 tailwind와 style scoped 활용

## 추가 기능 명세

- [x]  Modal 창을 이용하여 정보 출력
- [x]  검색 기능 추가

---

# 프로젝트 코드

## App.vue

```html
<template>
  <div class="p-6 max-w-5xl mx-auto">
    <h2 class="text-2xl font-bold mb-4 ml-4">📅 KCISA 문화데이터 캘린더</h2>
    <FestivalMain :setSelectedEvent="setSelectedEvent" />
    <BaseModal v-model="showModal">
      <div v-if="selectedEvent" class="flex gap-3 p-4 border rounded shadow">
        <img
          v-if="selectedEvent.image"
          :src="selectedEvent.image"
          alt="이벤트 이미지"
          class="w-auto h-[200px] object-cover mb-4 rounded"
        />
        <div>
          <h3 class="text-xl font-semibold mb-2">{{ selectedEvent.title }}</h3>
          <p>
            <strong>기간: </strong> {{ selectedEvent.start }} ~
            {{ selectedEvent.end }}
          </p>
          <p><strong>장소: </strong> {{ selectedEvent.place }}</p>
          <p>
            <strong>링크: </strong>
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

<style>
/* 마우스 올렸을 때 커서 추가 */
.fc-event {
  @apply cursor-pointer;
}
</style>

```

## pages/FestivalMain.vue

```html
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
    // 이벤트 DOM에 고유 ID 지정
    info.el.setAttribute("data-event-id", info.event.id);
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
  eventMouseEnter(info) {
    const eventId = info.event.id;
    const allEls = document.querySelectorAll(`[data-event-id="${eventId}"]`);
    allEls.forEach((el) => {
      el.style.backgroundColor = "rgba(0, 0, 0, 0.15)";
      el.style.borderColor = "rgba(0, 0, 0, 0.4)";
    });
  },
  eventMouseLeave(info) {
    const eventId = info.event.id;
    const { backgroundColor, borderColor } = info.event.extendedProps;
    const allEls = document.querySelectorAll(`[data-event-id="${eventId}"]`);
    allEls.forEach((el) => {
      el.style.backgroundColor = backgroundColor || "";
      el.style.borderColor = borderColor || "";
    });
  },
});

// 검색 버튼 클릭 or 엔터 시 호출
function onSearchClick() {
  const query = searchQuery.value.trim().toLowerCase();
  if (!query) {
    calendarOptions.value.events = [...allEvents.value];
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
        const bgColor = hexToRgba(eventColor[0], 0.3);
        const bdColor = hexToRgba(eventColor[0], 0.7);

        uniqueEventsMap.set(key, {
          id: key, // 반드시 고유 ID 지정
          title: item.TITLE,
          start,
          end,
          backgroundColor: bgColor,
          borderColor: bdColor,
          textColor: eventColor[1],
          extendedProps: {
            place: item.CNTC_INSTT_NM || "",
            url: item.URL,
            description: item.DESCRIPTION,
            image: item.IMAGE_OBJECT,
            backgroundColor: bgColor,
            borderColor: bdColor,
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

// 기간 파싱
function parsePeriod(period) {
  if (!period || !period.includes("~")) return { start: null, end: null };
  const [start, end] = period.split("~").map((s) => s.trim());
  return { start, end: end || start };
}

// HEX → RGBA 변환
function hexToRgba(hex, alpha = 0.4) {
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  return `rgba(${r}, ${g}, ${b}, ${alpha})`;
}

// 카테고리에 따른 색상
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

```

## component/BaseModal.vue

```html
<template>
  <div
    v-if="modelValue"
    class="fixed inset-0 bg-black/40 flex items-center justify-center z-50"
    @click.self="$emit('update:modelValue', false)"
  >
    <div
      class="relative bg-white rounded-lg shadow-lg w-[90%] max-h-[80vh] overflow-y-auto max-w-screen-md p-6"
    >
      <div class="flex justify-end absolute right-8 top-6">
        <button
          class="text-2xl"
          @click="$emit('update:modelValue', false)"
          aria-label="닫기"
        >
          &times;
        </button>
      </div>
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
defineProps({ modelValue: Boolean });
defineEmits(["update:modelValue"]);
</script>

```

---

# 프로젝트 결과물

## 1. GitHub 주소

[GitHub - et007693/Festival](https://github.com/et007693/Festival)

## 2. 내용

### 연도별 캘린더

![스크린샷 2025-05-21 오전 11.39.25.png](attachment:24755a2d-b06e-46b9-894f-e93976acc5e6:스크린샷_2025-05-21_오전_11.39.25.png)

### 월별 캘린더

![스크린샷 2025-05-21 오전 11.40.01.png](attachment:c3540799-45b2-40b5-a443-afa15b665a04:스크린샷_2025-05-21_오전_11.40.01.png)

### 주별 캘린더

![스크린샷 2025-05-21 오전 11.40.32.png](attachment:7aabe19b-2267-42eb-8524-117b255236a2:스크린샷_2025-05-21_오전_11.40.32.png)

### 전시 일정 클릭 시, 모달 창 팝업

![스크린샷 2025-05-21 오전 11.40.19.png](attachment:6a62a689-1ae3-4782-806f-9163fd97b596:스크린샷_2025-05-21_오전_11.40.19.png)

### 검색 기능

![스크린샷 2025-05-21 오전 11.43.31.png](attachment:280dee9a-4d1e-4dd8-85ac-3cbc53bb1000:스크린샷_2025-05-21_오전_11.43.31.png)

---

# 사용 API 및 라이브러리

## 1. 문화체육관광부 소속 기관 전시 정보 데이터

[문화공공데이터광장-맞춤형 API 상세](https://www.culture.go.kr/data/openapi/openapiView.do?id=598)

## 2. FullCalendar 라이브러리

[FullCalendar - JavaScript Event Calendar](https://fullcalendar.io/)

---

# Getting Start

## yarn

```bash
yarn install
yarn add @fullcalendar/vue3 @fullcalendar/core @fullcalendar/daygrid @fullcalendar/interaction
```

## npm

```bash
yarn install
npm install --save \
  @fullcalendar/core \
  @fullcalendar/vue3
```
