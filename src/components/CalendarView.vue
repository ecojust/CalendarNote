<template>
  <div class="calendar-container">
    <div class="titlebar" data-tauri-drag-region>
      <div class="titlebar-title">CalendarNote</div>
      <div class="titlebar-controls">
        <button
          class="titlebar-btn"
          id="titlebar-minimize"
          @click="minimize"
          data-tauri-drag-region="false"
        >
          <svg width="12" height="12" viewBox="0 0 12 12">
            <path d="M2 6h8" stroke="currentColor" stroke-width="1.5" />
          </svg>
        </button>
        <button
          class="titlebar-btn"
          id="titlebar-maximize"
          @click="toggleMaximize"
          data-tauri-drag-region="false"
        >
          <svg v-if="!isMaximized" width="12" height="12" viewBox="0 0 12 12">
            <rect
              x="2"
              y="2"
              width="8"
              height="8"
              stroke="currentColor"
              stroke-width="1.5"
              fill="none"
            />
          </svg>
          <svg v-else width="12" height="12" viewBox="0 0 12 12">
            <path
              d="M2 4h6v6H2z"
              stroke="currentColor"
              stroke-width="1.5"
              fill="none"
            />
            <path
              d="M4 2h6v6"
              stroke="currentColor"
              stroke-width="1.5"
              fill="none"
            />
          </svg>
        </button>
        <button
          class="titlebar-btn"
          id="titlebar-close"
          @click="close"
          data-tauri-drag-region="false"
        >
          <svg width="12" height="12" viewBox="0 0 12 12">
            <path
              d="M2 2l8 8M10 2l-8 8"
              stroke="currentColor"
              stroke-width="1.5"
            />
          </svg>
        </button>
      </div>
    </div>

    <FullCalendar ref="fullCalendarRef" :options="calendarOptions" />

    <!-- 右键菜单 -->
    <div
      v-if="contextMenu.visible"
      class="context-menu"
      :style="{ left: contextMenu.x + 'px', top: contextMenu.y + 'px' }"
    >
      <div class="context-menu-item" @click="openAddNoteDialog">
        <svg width="14" height="14" viewBox="0 0 14 14">
          <path
            d="M7 2V12M2 7H12"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
          />
        </svg>
        添加便签
      </div>
    </div>

    <!-- 添加便签弹窗 -->
    <div v-if="showAddDialog" class="dialog-overlay" @click="closeAddDialog">
      <div class="dialog" @click.stop>
        <div class="dialog-header">
          <h3>添加便签</h3>
          <button class="dialog-close" @click="closeAddDialog">
            <svg width="14" height="14" viewBox="0 0 14 14">
              <path
                d="M2 2l10 10M12 2l-10 10"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
              />
            </svg>
          </button>
        </div>
        <div class="dialog-body">
          <div class="form-group">
            <label>标题 <span class="required">*</span></label>
            <input
              v-model="newNote.title"
              type="text"
              placeholder="请输入标题"
            />
          </div>
          <div class="form-group">
            <label>内容</label>
            <textarea
              v-model="newNote.content"
              placeholder="请输入内容（可选）"
              rows="3"
            ></textarea>
          </div>
          <div class="form-group">
            <label>提醒时间</label>
            <input v-model="newNote.reminder" type="datetime-local" />
          </div>
          <div class="form-group">
            <label>颜色</label>
            <div class="color-options">
              <div
                v-for="color in colors"
                :key="color"
                class="color-option"
                :class="{ active: newNote.color === color }"
                :style="{ backgroundColor: color }"
                @click="newNote.color = color"
              ></div>
            </div>
          </div>
        </div>
        <div class="dialog-footer">
          <button class="btn-cancel" @click="closeAddDialog">取消</button>
          <button
            class="btn-confirm"
            @click="addNote"
            :disabled="!newNote.title.trim()"
          >
            确定
          </button>
        </div>
      </div>
    </div>

    <!-- 便签详情弹窗 -->
    <div
      v-if="showDetailDialog && selectedNote"
      class="dialog-overlay"
      @click="closeDetailDialog"
    >
      <div class="dialog" @click.stop>
        <div class="dialog-header">
          <h3>便签详情</h3>
          <button class="dialog-close" @click="closeDetailDialog">
            <svg width="14" height="14" viewBox="0 0 14 14">
              <path
                d="M2 2l10 10M12 2l-10 10"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
              />
            </svg>
          </button>
        </div>
        <div class="dialog-body">
          <div class="detail-item">
            <label>标题</label>
            <div class="detail-value">{{ selectedNote.title }}</div>
          </div>
          <div v-if="selectedNote.content" class="detail-item">
            <label>内容</label>
            <div class="detail-value detail-content">
              {{ selectedNote.content }}
            </div>
          </div>
          <div class="detail-item">
            <label>日期</label>
            <div class="detail-value">{{ selectedNote.date }}</div>
          </div>
          <div v-if="selectedNote.reminder" class="detail-item">
            <label>提醒时间</label>
            <div class="detail-value">{{ selectedNote.reminder }}</div>
          </div>
          <div class="detail-item">
            <label>颜色</label>
            <div
              class="detail-color"
              :style="{ backgroundColor: selectedNote.color }"
            ></div>
          </div>
        </div>
        <div class="dialog-footer">
          <button class="btn-cancel" @click="closeDetailDialog">关闭</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import type { EventInput } from "@fullcalendar/core";
import { Solar, HolidayUtil } from "lunar-typescript";

interface Note {
  id: string;
  title: string;
  content: string;
  date: string;
  color: string;
  reminder?: string;
  created_at: string;
}

const props = defineProps<{
  notes: Note[];
}>();

const emit = defineEmits<{
  (e: "add-note", note: Note): void;
  (e: "delete-note", id: string): void;
  (e: "move-note", id: string, newDate: string): void;
}>();

const fullCalendarRef = ref<InstanceType<typeof FullCalendar>>();
const isMaximized = ref(false);

const contextMenu = ref({
  visible: false,
  x: 0,
  y: 0,
  date: "",
});

const showAddDialog = ref(false);
const showDetailDialog = ref(false);
const selectedNote = ref<Note | null>(null);
const newNote = ref({
  title: "",
  content: "",
  color: "#ff9a9e",
  reminder: "",
});

const colors = [
  "#ff9a9e",
  "#fad0c4",
  "#ffecd2",
  "#fcb69f",
  "#a18cd1",
  "#fbc2eb",
  "#a6c1ee",
  "#84fab0",
];

const calendarEvents = computed<EventInput[]>(() => {
  return props.notes.map((note) => ({
    id: note.id,
    title: note.title,
    start: note.date,
    backgroundColor: note.color,
    borderColor: note.color,
    extendedProps: {
      content: note.content,
      reminder: note.reminder,
    },
  }));
});

const calendarOptions = {
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  locale: "zh-cn",
  firstDay: 0,
  height: "100%",
  headerToolbar: {
    left: "prev,next today",
    center: "title",
    right: "",
  },
  buttonText: {
    today: "今天",
    prev: "<",
    next: ">",
  },
  dayMaxEvents: 2,
  editable: true,
  selectable: true,
  selectMirror: true,
  dayCellContent: handleDayCellContent,
  eventDrop: handleEventDrop,
  eventClick: handleEventClick,
  eventDidMount: handleEventMount,
  datesSet: () => {},
} as any;

watch(
  () => props.notes.length,
  () => {
    const calendarApi = fullCalendarRef.value?.getApi();
    if (calendarApi) {
      calendarApi.removeAllEvents();
      calendarApi.addEventSource(calendarEvents.value);
    }
  },
);

function handleContextMenu(event: Event) {
  const mouseEvent = event as MouseEvent;
  const target = mouseEvent.target as HTMLElement;
  const dayCell = target.closest(".fc-daygrid-day");
  if (dayCell) {
    const dateAttr = dayCell.getAttribute("data-date");
    if (dateAttr) {
      event.preventDefault();
      contextMenu.value = {
        visible: true,
        x: mouseEvent.clientX,
        y: mouseEvent.clientY,
        date: dateAttr,
      };
    }
  }
}

function handleDayCellContent(arg: any) {
  const solar = Solar.fromYmd(
    arg.date.getFullYear(),
    arg.date.getMonth() + 1,
    arg.date.getDate(),
  );
  const lunar = solar.getLunar();

  const jieQi = lunar.getJieQi();
  const festivals = [...lunar.getFestivals(), ...solar.getFestivals()];

  let bottomText = "";
  if (jieQi) {
    bottomText = jieQi;
  } else if (festivals.length > 0) {
    bottomText = festivals[0];
  } else if (lunar.getDay() === 1) {
    bottomText = lunar.getMonthInChinese();
  } else {
    bottomText = lunar.getDayInChinese();
  }

  const holiday = HolidayUtil.getHoliday(solar.toYmd());
  const tag =
    holiday && holiday.isWork()
      ? '<span class="fc-day-holiday-tag work">班</span>'
      : holiday
        ? '<span class="fc-day-holiday-tag rest">休</span>'
        : "";

  const extra =
    jieQi || festivals.length > 0
      ? '<span class="fc-day-lunar festival">' + bottomText + "</span>"
      : '<span class="fc-day-lunar">' + bottomText + "</span>";

  return {
    html:
      '<div class="fc-daygrid-day-number">' +
      arg.dayNumberText +
      "</div>" +
      extra +
      tag,
  };
}

function handleEventDrop(info: any) {
  const noteId = info.event.id;
  const newDate = info.event.startStr;
  const isCopy = info.jsEvent?.ctrlKey || info.jsEvent?.metaKey;

  console.log(
    "[FullCalendar] eventDrop:",
    noteId,
    "->",
    newDate,
    isCopy ? "(copy)" : "(move)",
  );

  if (isCopy) {
    const originalNote = props.notes.find((n) => n.id === noteId);
    if (originalNote) {
      const newNote: Note = {
        id: Date.now().toString() + Math.random().toString(36).substr(2, 9),
        title: originalNote.title,
        content: originalNote.content,
        date: newDate,
        color: originalNote.color,
        reminder: originalNote.reminder,
        created_at: new Date().toISOString(),
      };
      emit("add-note", newNote);
      const calendarApi = fullCalendarRef.value?.getApi();
      if (calendarApi) {
        info.revert();
      }
    }
  } else {
    emit("move-note", noteId, newDate);
  }
}

function handleEventClick(info: any) {
  const noteId = info.event.id;
  const note = props.notes.find((n) => n.id === noteId);
  if (note) {
    console.log("[FullCalendar] eventClick:", note);
  }
}

function handleEventMount(info: any) {
  const noteId = info.event.id;
  const note = props.notes.find((n) => n.id === noteId);
  if (note && note.content) {
    info.el.title = note.content;
  }

  const deleteBtn = document.createElement("div");
  deleteBtn.className = "event-delete-btn";
  deleteBtn.innerHTML = "×";
  deleteBtn.addEventListener("click", (e) => {
    e.stopPropagation();
    emit("delete-note", info.event.id);
  });
  info.el.style.position = "relative";
  info.el.appendChild(deleteBtn);

  info.el.addEventListener("dblclick", () => {
    const noteData = props.notes.find((n) => n.id === info.event.id);
    if (noteData) {
      selectedNote.value = noteData;
      showDetailDialog.value = true;
    }
  });
}

function openAddNoteDialog() {
  showAddDialog.value = true;
  hideContextMenu();
}

function closeAddDialog() {
  showAddDialog.value = false;
  newNote.value = {
    title: "",
    content: "",
    color: "#ff9a9e",
    reminder: "",
  };
}

function closeDetailDialog() {
  showDetailDialog.value = false;
  selectedNote.value = null;
}

function hideContextMenu() {
  contextMenu.value.visible = false;
}

function addNote() {
  if (!newNote.value.title.trim()) return;

  const note: Note = {
    id: Date.now().toString() + Math.random().toString(36).substr(2, 9),
    title: newNote.value.title.trim(),
    content: newNote.value.content.trim(),
    date: contextMenu.value.date,
    color: newNote.value.color,
    reminder: newNote.value.reminder || undefined,
    created_at: new Date().toISOString(),
  };

  emit("add-note", note);
  closeAddDialog();
}

async function minimize() {
  const { getCurrentWindow } = await import("@tauri-apps/api/window");
  await getCurrentWindow().minimize();
}

async function toggleMaximize() {
  const { getCurrentWindow } = await import("@tauri-apps/api/window");
  await getCurrentWindow().toggleMaximize();
  isMaximized.value = !isMaximized.value;
}

async function close() {
  const { getCurrentWindow } = await import("@tauri-apps/api/window");
  await getCurrentWindow().close();
}

onMounted(() => {
  document.addEventListener("click", hideContextMenu);
  const calendarEl = document.querySelector(".fc");
  if (calendarEl) {
    calendarEl.addEventListener("contextmenu", handleContextMenu);
  }
});

onUnmounted(() => {
  document.removeEventListener("click", hideContextMenu);
  const calendarEl = document.querySelector(".fc");
  if (calendarEl) {
    calendarEl.removeEventListener("contextmenu", handleContextMenu);
  }
});
</script>

<style lang="less">
.calendar-container {
  position: relative;
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 0;
  background: transparent;
  overflow: hidden;
  border: none;
}

.calendar-container::before {
  content: "";
  position: absolute;
  top: -20px;
  left: -20px;
  right: -20px;
  bottom: -20px;
  background: url(https://picsum.photos/1920/1080?random=1) center/cover
    no-repeat fixed;
  opacity: 1.9;
  filter: blur(20px);
  border-radius: 20px;
  z-index: -1;
}

.titlebar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 36px;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.06),
    rgba(255, 255, 255, 0)
  );
  border-bottom: 2px solid rgba(255, 255, 255, 0.8);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.08);
  user-select: none;
  flex-shrink: 0;
}

.titlebar-title {
  padding: 0 16px;
  color: white;
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 0.5px;
}

.titlebar-controls {
  display: flex;
}

.titlebar-btn {
  appearance: none;
  padding: 0;
  margin: 0;
  border: none;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: 36px;
  height: 36px;
  background-color: transparent;
  color: white;
  cursor: pointer;

  &:hover {
    background: rgba(255, 255, 255, 0.15);
  }

  &#titlebar-close:hover {
    background: #ff6b6b;
    color: white;
  }
}

/* FullCalendar 主题覆盖 */
.fc {
  flex: 1;
  min-height: 0;
  background: transparent;
  display: flex;
  flex-direction: column;
  * {
    border-width: 0 !important;
  }
}

.fc .fc-scrollgrid > tbody .fc-scrollgrid-section > td {
  border-bottom-color: transparent !important;
  box-shadow: none !important;
}

.fc .fc-toolbar-title {
  font-size: 18px;
  font-weight: 700;
  color: white;
  text-shadow: 0 2px 8px rgba(214, 51, 132, 0.3);
}

.fc .fc-button {
  background: transparent;
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
  border-radius: 20px;
  padding: 4px 14px;
  font-weight: 600;
  font-size: 12px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &:hover {
    background: rgba(255, 255, 255, 0.15);
    transform: scale(1.05);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  }

  &:active {
    transform: scale(0.95);
  }

  &.fc-button-active {
    background: rgba(255, 255, 255, 0.2);
    border-color: rgba(255, 255, 255, 0.4);
    color: white;
  }
}

.fc .fc-toolbar {
  padding: 12px 16px 8px;
  gap: 8px;
}

.fc .fc-toolbar-chunk {
  display: flex;
  gap: 6px;
}

.fc .fc-view-harness {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
}

.fc .fc-view {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
}

.fc .fc-scrollgrid {
  border: none !important;
  border-bottom: none !important;
  border-color: transparent !important;
  box-shadow: none !important;
  border-radius: 12px;
  overflow: hidden;
  flex: 1;
  min-height: 0;
}

.fc .fc-daygrid-body {
  flex: 1;
  min-height: 0;
}

.fc .fc-daygrid-body table {
  height: 100%;
}

.fc .fc-daygrid-row {
  flex: 1;
  min-height: 0;
}

.fc .fc-daygrid-row-table {
  height: 100%;
}

.fc .fc-daygrid-row-cells {
  height: 100%;
}

.fc .fc-daygrid-row-cells td {
  height: 100%;
}
.fc-daygrid-day-top {
  align-items: center;
  justify-content: center;
  a {
    display: block;
  }
}
.fc .fc-col-header-cell {
  background: transparent;
  border-width: 2px !important;
  padding: 8px 0;

  .fc-col-header-cell-cushion {
    display: block;
    color: rgba(255, 255, 255, 0.8);
    font-weight: 600;
    font-size: 11px;
    letter-spacing: 1px;
    text-transform: uppercase;
    padding: 4px 0;
    text-align: center;
  }
}

.fc .fc-daygrid-day {
  background: transparent;
  border-width: 2px !important;
  transition: background 0.2s ease;
  cursor: pointer;

  &:hover {
    background: rgba(255, 255, 255, 0.1);
  }
}

.fc .fc-daygrid-day.fc-day-today {
  background: transparent;
  // border: 2px solid #ff6b9d;
  box-shadow: 0 0 20px 3px rgba(255, 107, 157, 0.92) inset;
}

.fc .fc-daygrid-day-number {
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-weight: 600;
  font-size: 12px;
  padding: 6px 8px 2px;
  text-align: center;

  &:hover {
    color: #ff6b9d;
  }
}

.fc-day-lunar {
  display: block;
  text-align: center;
  font-size: 9px;
  color: rgba(255, 255, 255, 0.6);
  padding: 0 4px 2px;
  line-height: 1.2;
  overflow: hidden;
  white-space: nowrap;

  &.festival {
    color: #ffdce8;
    font-weight: 600;
  }
}

.fc-day-holiday-tag {
  display: inline-block;
  margin: 1px auto;
  padding: 0 4px;
  border-radius: 3px;
  font-size: 9px;
  line-height: 14px;
  color: white;
  font-weight: 600;
  vertical-align: top;

  &.rest {
    background: rgba(255, 107, 157, 0.85);
  }

  &.work {
    background: rgba(79, 172, 254, 0.85);
  }
}

.fc .fc-daygrid-day.fc-day-other {
  opacity: 0.5;
}

.fc .fc-daygrid-day.fc-day-other .fc-daygrid-day-number {
  color: white;
}

.fc .fc-daygrid-event {
  border-radius: 6px !important;
  border: none !important;
  padding: 2px 6px !important;
  margin: 1px 3px !important;
  font-size: 10px !important;
  font-weight: 500;
  color: white !important;
  cursor: grab !important;
  transition: box-shadow 0.2s ease;
}

.fc .fc-daygrid-event-harness {
  margin-top: 1px;
}

.fc .fc-event {
  cursor: grab !important;
}

.fc .fc-event:active {
  cursor: grabbing !important;
}

.fc .fc-more-link {
  color: white !important;
  font-weight: 600;
  font-size: 10px;
  padding: 2px 6px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.15);

  &:hover {
    background: rgba(255, 255, 255, 0.25);
  }
}

.event-delete-btn {
  position: absolute;
  top: -4px;
  right: -4px;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #ff6b6b;
  color: white;
  font-size: 10px;
  line-height: 14px;
  text-align: center;
  cursor: pointer;
  opacity: 0;
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;

  &:hover {
    background: #ff4757;
    transform: scale(1.1);
  }
}

.fc .fc-daygrid-event:hover .event-delete-btn {
  opacity: 1;
}

.fc .fc-highlight {
  background: rgba(255, 255, 255, 0.1) !important;
  border-radius: 8px;
}

.fc .fc-daygrid-bg-harness {
  border-radius: 8px;
}

/* 右键菜单 */
.context-menu {
  position: fixed;
  background: rgba(255, 255, 255, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
  padding: 6px;
  z-index: 1000;
  min-width: 140px;
}

.context-menu-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  color: white;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.15);
  }
}

/* 弹窗 */
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.dialog {
  background: rgba(255, 255, 255, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  width: 360px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  animation: dialogIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes dialogIn {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);

  h3 {
    font-size: 16px;
    font-weight: 700;
    color: white;
    margin: 0;
  }
}

.dialog-close {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  padding: 4px;
  border-radius: 6px;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.15);
    color: white;
  }
}

.dialog-body {
  padding: 20px;
}

.form-group {
  margin-bottom: 16px;

  &:last-child {
    margin-bottom: 0;
  }

  label {
    display: block;
    font-size: 12px;
    font-weight: 600;
    color: rgba(255, 255, 255, 0.8);
    margin-bottom: 6px;

    .required {
      color: #ff6b9d;
    }
  }

  input,
  textarea {
    width: 100%;
    padding: 10px 12px;
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-radius: 10px;
    font-size: 13px;
    color: white;
    background: transparent;
    transition: all 0.2s ease;
    box-sizing: border-box;

    &:focus {
      outline: none;
      border-color: #ff9a9e;
      box-shadow: 0 0 0 3px rgba(255, 154, 158, 0.15);
    }

    &::placeholder {
      color: rgba(255, 255, 255, 0.5);
    }
  }

  textarea {
    resize: none;
  }
}

.color-options {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.color-option {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;
  border: 2px solid transparent;

  &:hover {
    transform: scale(1.15);
  }

  &.active {
    border-color: white;
    box-shadow: 0 0 0 2px #5a4a6a;
    transform: scale(1.1);
  }
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 16px 20px;
  border-top: 1px solid rgba(255, 255, 255, 0.4);
}

.btn-cancel,
.btn-confirm {
  padding: 8px 20px;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-cancel {
  background: transparent;
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;

  &:hover {
    background: rgba(255, 255, 255, 0.15);
  }
}

.btn-confirm {
  background: linear-gradient(135deg, #ff9a9e 0%, #d63384 100%);
  border: none;
  color: white;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.3);

  &:hover:not(:disabled) {
    transform: translateY(-1px);
    box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.detail-item {
  margin-bottom: 16px;

  &:last-child {
    margin-bottom: 0;
  }

  label {
    display: block;
    font-size: 12px;
    font-weight: 600;
    color: rgba(255, 255, 255, 0.8);
    margin-bottom: 6px;
  }
}

.detail-value {
  font-size: 14px;
  color: white;
  line-height: 1.5;
}

.detail-content {
  white-space: pre-wrap;
  word-break: break-word;
  background: transparent;
  padding: 10px 12px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.detail-color {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  border: 2px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 0 0 2px rgba(255, 255, 255, 0.3);
}
</style>
