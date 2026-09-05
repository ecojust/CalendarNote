<template>
  <div
    class="calendar-container"
    :style="{ '--bg-blur': bgBlur + 'px', '--bg-image': `url('${wallpaperBg}')` }"
  >
    <div class="titlebar" :data-tauri-drag-region="isLocked ? 'false' : 'true'">
      <div class="titlebar-title">CalendarNote</div>
      <div class="titlebar-controls">
        <button
          class="titlebar-btn"
          id="titlebar-pin"
          :class="{ locked: isLocked }"
          :title="isLocked ? '解锁窗口' : '锁定窗口'"
          @click="toggleLock"
          data-tauri-drag-region="false"
        >
          <svg
            v-if="!isLocked"
            width="14"
            height="14"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="1.8"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <line x1="12" x2="12" y1="17" y2="22" />
            <path
              d="M5 17h14v-1.76a2 2 0 0 0-1.11-1.79l-1.78-.9A2 2 0 0 1 15 10.76V6h1a2 2 0 0 0 0-4H8a2 2 0 0 0 0 4h1v4.76a2 2 0 0 1-1.11 1.79l-1.78.9A2 2 0 0 0 5 15.24Z"
            />
          </svg>
          <svg
            v-else
            width="14"
            height="14"
            viewBox="0 0 24 24"
            fill="currentColor"
            stroke="currentColor"
            stroke-width="1.8"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path
              d="M5 17h14v-1.76a2 2 0 0 0-1.11-1.79l-1.78-.9A2 2 0 0 1 15 10.76V6h1a2 2 0 0 0 0-4H8a2 2 0 0 0 0 4h1v4.76a2 2 0 0 1-1.11 1.79l-1.78.9A2 2 0 0 0 5 15.24Z"
            />
            <line x1="12" x2="12" y1="17" y2="22" />
          </svg>
        </button>
        <button
          class="titlebar-btn"
          id="titlebar-settings"
          title="设置"
          @click="openSettingsDialog"
          data-tauri-drag-region="false"
        >
          <svg
            width="14"
            height="14"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="1.8"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path
              d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"
            />
            <circle cx="12" cy="12" r="3" />
          </svg>
        </button>
        <!-- 最小化/最大化已暂禁用
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
        -->
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

    <div class="calendar-toolbar">
      <button class="toolbar-btn" title="上一月" @click="prevMonth">
        <svg
          width="16"
          height="16"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polyline points="15 18 9 12 15 6" />
        </svg>
      </button>
      <div class="toolbar-title">{{ currentTitle }}</div>
      <button class="toolbar-btn" title="下一月" @click="nextMonth">
        <svg
          width="16"
          height="16"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polyline points="9 18 15 12 9 6" />
        </svg>
      </button>
      <button class="toolbar-btn" title="回到今天" @click="goToday">
        <svg
          width="16"
          height="16"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
        >
          <circle cx="12" cy="12" r="10" />
          <line x1="12" y1="4" x2="12" y2="8" />
          <line x1="12" y1="16" x2="12" y2="20" />
          <line x1="4" y1="12" x2="8" y2="12" />
          <line x1="16" y1="12" x2="20" y2="12" />
          <circle cx="12" cy="12" r="2" />
        </svg>
      </button>
      <button
        class="toolbar-btn"
        :class="{ danger: clearConfirm }"
        :title="
          clearConfirm ? '再次点击确认清除当月便签' : '清除当前月所有便签'
        "
        @click="toggleClearMonth"
      >
        <svg
          v-if="!clearConfirm"
          width="16"
          height="16"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <path d="M3 6h18" />
          <path d="M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2" />
          <path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6" />
        </svg>
        <svg
          v-else
          width="16"
          height="16"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polyline points="20 6 9 17 4 12" />
        </svg>
      </button>
    </div>

    <FullCalendar ref="fullCalendarRef" :options="calendarOptions" />

    <!-- 右键菜单 -->
    <div
      v-if="contextMenu.visible"
      class="context-menu"
      :style="{ left: contextMenu.x + 'px', top: contextMenu.y + 'px' }"
    >
      <div class="context-menu-item" @click="openAddNoteDialog">
        <svg width="12" height="12" viewBox="0 0 12 12">
          <path
            d="M6 1v10M1 6h10"
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
            <div class="color-options">
              <div
                v-for="color in colors"
                :key="color"
                class="color-option"
                :class="{ active: selectedNote.color === color }"
                :style="{ backgroundColor: color }"
                @click="updateNoteColor(color)"
              ></div>
            </div>
          </div>
        </div>
        <div class="dialog-footer">
          <button class="btn-cancel" @click="closeDetailDialog">关闭</button>
        </div>
      </div>
    </div>

    <!-- 设置弹窗 -->
    <div
      v-if="showSettingsDialog"
      class="dialog-overlay"
      @click="closeSettingsDialog"
    >
      <div class="dialog" @click.stop>
        <div class="dialog-header">
          <h3>设置</h3>
          <button class="dialog-close" @click="closeSettingsDialog">
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
            <label>壁纸</label>
            <div class="wallpaper-row">
              <div
                class="wallpaper-preview"
                :style="{ backgroundImage: `url('${wallpaperBg}')` }"
              ></div>
              <div class="wallpaper-actions">
                <button
                  class="btn-mini"
                  :disabled="updatingWallpaper"
                  @click="updateWallpaper"
                >
                  {{ updatingWallpaper ? "下载中…" : "更新壁纸" }}
                </button>
              </div>
            </div>
          </div>
          <div class="form-group">
            <label>背景模糊度（{{ bgBlur }}px）</label>
            <input
              v-model.number="bgBlur"
              type="range"
              min="0"
              max="40"
              step="1"
              class="blur-slider"
            />
          </div>
        </div>
        <div class="dialog-footer">
          <button class="btn-cancel" @click="bgBlur = 20">恢复默认</button>
          <button class="btn-confirm" @click="closeSettingsDialog">完成</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from "vue";
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
  config?: Record<string, string>;
}>();

const emit = defineEmits<{
  (e: "add-note", note: Note): void;
  (e: "delete-note", id: string): void;
  (e: "move-note", id: string, newDate: string): void;
  (e: "update-note", id: string, patch: Partial<Note>): void;
  (e: "month-change", start: string, end: string): void;
  (e: "clear-month", start: string, end: string): void;
  (e: "update-config", key: string, value: string): void;
}>();

const fullCalendarRef = ref<InstanceType<typeof FullCalendar>>();
const isLocked = ref(localStorage.getItem("calendar-locked") === "1");
const currentTitle = ref("");
const currentRange = ref({ start: "", end: "" });
const clearConfirm = ref(false);
const showSettingsDialog = ref(false);
const bgBlur = ref(
  props.config?.["bg-blur"] != null
    ? Number(props.config["bg-blur"])
    : Number(localStorage.getItem("calendar-bg-blur") ?? 20),
);
const defaultWallpaper = "https://picsum.photos/1920/1080?random=1";
const wallpaper = ref(
  props.config?.["wallpaper"] != null
    ? props.config["wallpaper"]
    : localStorage.getItem("calendar-wallpaper") ?? "",
);
const wallpaperBg = ref(defaultWallpaper);
const updatingWallpaper = ref(false);

async function applyWallpaper(pathOrData: string) {
  if (!pathOrData) {
    wallpaperBg.value = defaultWallpaper;
    return;
  }
  if (pathOrData.startsWith("data:") || pathOrData.startsWith("http")) {
    wallpaperBg.value = pathOrData;
    return;
  }
  // @ts-ignore
  if (window.__TAURI__) {
    try {
      const { convertFileSrc } = await import("@tauri-apps/api/core");
      wallpaperBg.value = convertFileSrc(pathOrData);
    } catch (e) {
      console.error("Failed to load wallpaper:", e);
      wallpaperBg.value = defaultWallpaper;
    }
  } else {
    wallpaperBg.value = defaultWallpaper;
  }
}

function readFileAsDataURL(blob: Blob): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result as string);
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
}

async function updateWallpaper() {
  if (updatingWallpaper.value) return;
  updatingWallpaper.value = true;
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const { invoke } = await import("@tauri-apps/api/core");
      wallpaper.value = await invoke<string>("download_wallpaper", {
        url: defaultWallpaper,
      });
    } else {
      const res = await fetch(defaultWallpaper);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      wallpaper.value = await readFileAsDataURL(await res.blob());
    }
  } catch (e) {
    console.error("Failed to update wallpaper:", e);
  } finally {
    updatingWallpaper.value = false;
  }
}

watch(bgBlur, (value) => {
  localStorage.setItem("calendar-bg-blur", String(value));
  emit("update-config", "bg-blur", String(value));
});

watch(wallpaper, (value) => {
  localStorage.setItem("calendar-wallpaper", value);
  applyWallpaper(value);
  emit("update-config", "wallpaper", value);
});

watch(
  () => props.config,
  (cfg) => {
    if (cfg?.["bg-blur"] != null) {
      bgBlur.value = Number(cfg["bg-blur"]);
    }
    if (cfg?.["wallpaper"] != null) {
      wallpaper.value = cfg["wallpaper"];
    }
  },
  { immediate: true },
);

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
  color: "#ff5f8f",
  reminder: "",
});

const colors = [
  "#ff5f8f",
  "#ff8a5c",
  "#ffc53d",
  "#ff6fb0",
  "#7c5cff",
  "#3dd6ff",
  "#3ce0a8",
  "#ff4d6d",
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
    left: "",
    center: "",
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
  datesSet: handleDatesSet,
} as any;

watch(
  () => props.notes,
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
      openContextMenu(mouseEvent.clientX, mouseEvent.clientY, dateAttr);
    }
  }
}

function openContextMenu(x: number, y: number, date: string) {
  contextMenu.value = { visible: true, x, y, date };
  nextTick(() => {
    const el = document.querySelector<HTMLElement>(".context-menu");
    if (!el) return;
    const rect = el.getBoundingClientRect();
    let nextX = x;
    let nextY = y;
    if (nextX + rect.width > window.innerWidth - 8) {
      nextX = window.innerWidth - rect.width - 8;
    }
    if (nextY + rect.height > window.innerHeight - 8) {
      nextY = window.innerHeight - rect.height - 8;
    }
    if (nextX !== x || nextY !== y) {
      contextMenu.value = { ...contextMenu.value, x: nextX, y: nextY };
    }
  });
}

function handleDatesSet(arg: any) {
  const d = arg.view.currentStart;
  currentTitle.value = `${d.getFullYear()}年${d.getMonth() + 1}月`;
  currentRange.value = {
    start: formatYmd(arg.view.currentStart),
    end: formatYmd(arg.view.currentEnd),
  };
  clearConfirm.value = false;
  emit("month-change", currentRange.value.start, currentRange.value.end);
}

function toggleClearMonth() {
  if (!clearConfirm.value) {
    clearConfirm.value = true;
    window.setTimeout(() => {
      clearConfirm.value = false;
    }, 3000);
  } else {
    emit("clear-month", currentRange.value.start, currentRange.value.end);
    clearConfirm.value = false;
  }
}

function formatYmd(date: Date) {
  const y = date.getFullYear();
  const m = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");
  return `${y}-${m}-${day}`;
}

function prevMonth() {
  fullCalendarRef.value?.getApi().prev();
}

function nextMonth() {
  fullCalendarRef.value?.getApi().next();
}

function goToday() {
  fullCalendarRef.value?.getApi().today();
}

const dayCellCache = new Map<string, string>();

function handleDayCellContent(arg: any) {
  const y = arg.date.getFullYear();
  const m = arg.date.getMonth() + 1;
  const d = arg.date.getDate();
  const key = `${y}-${m}-${d}`;
  const cached = dayCellCache.get(key);
  if (cached) {
    return { html: cached };
  }

  const solar = Solar.fromYmd(y, m, d);
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

  const html =
    '<div class="fc-daygrid-day-number">' +
    arg.dayNumberText +
    "</div>" +
    extra +
    tag;

  dayCellCache.set(key, html);
  return { html };
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
  deleteBtn.innerHTML =
    '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 8 8" width="8" height="8" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"><path d="M1.5 1.5l5 5M6.5 1.5l-5 5"/></svg>';
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

function openSettingsDialog() {
  showSettingsDialog.value = true;
}

function closeSettingsDialog() {
  showSettingsDialog.value = false;
}

function updateNoteColor(color: string) {
  if (!selectedNote.value) return;
  selectedNote.value.color = color;
  emit("update-note", selectedNote.value.id, { color });
}

function closeAddDialog() {
  showAddDialog.value = false;
  newNote.value = {
    title: "",
    content: "",
    color: "#ff5f8f",
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

// 最小化/最大化已暂禁用（按钮已注释）
// async function minimize() {
//   const { getCurrentWindow } = await import("@tauri-apps/api/window");
//   await getCurrentWindow().minimize();
// }
//
// async function toggleMaximize() {
//   const { getCurrentWindow } = await import("@tauri-apps/api/window");
//   await getCurrentWindow().toggleMaximize();
//   isMaximized.value = !isMaximized.value;
// }

function toggleLock() {
  isLocked.value = !isLocked.value;
  localStorage.setItem("calendar-locked", isLocked.value ? "1" : "0");
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
  background: var(--bg-image, url(https://picsum.photos/1920/1080?random=1))
    center/cover no-repeat fixed;
  opacity: 1.9;
  filter: blur(var(--bg-blur, 20px));
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

/* 自定义日历 toolbar */
.calendar-toolbar {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  padding: 12px 16px 8px;

  flex-shrink: 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.fc .fc-header-toolbar {
  display: none;
}

.fc .fc-col-header {
  border-top: 20px solid red;
}

.toolbar-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  padding: 0;
  border: none;
  background: transparent;
  color: white;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.15);
  }

  &.danger {
    color: #ff5f5f;
    background: rgba(255, 95, 95, 0.25);
    animation: dangerPulse 0.8s ease-in-out infinite alternate;
  }

  &:active {
    transform: scale(0.85);
  }
}

@keyframes dangerPulse {
  from {
    box-shadow: 0 0 0 rgba(255, 95, 95, 0.4);
  }
  to {
    box-shadow: 0 0 12px rgba(255, 95, 95, 0.9);
  }
}

.toolbar-title {
  min-width: 120px;
  text-align: center;
  font-size: 18px;
  font-weight: 700;
  color: white;
  text-shadow: 0 2px 8px rgba(214, 51, 132, 0.3);
  user-select: none;
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
  box-shadow: 0px 0 20px 3px rgba(107, 179, 255, 0.92) inset;
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
  border-radius: 999px !important;
  border: 1px solid rgba(255, 255, 255, 0.6) !important;
  padding: 1px 6px !important;
  margin: 1px 4px !important;
  font-size: 10px !important;
  line-height: 14px !important;
  font-weight: 700;
  color: white !important;
  cursor: grab !important;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.18);
  transition: box-shadow 0.2s ease;

  &:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.28);
  }

  &.fc-event-dragging {
    opacity: 0.85;
  }

  .fc-event-main {
    color: white !important;
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.25);
  }

  .fc-event-time {
    display: none;
  }
}

.fc .fc-daygrid-event .fc-event-title::before {
  content: "✦";
  margin-right: 4px;
  font-size: 8px;
  opacity: 0.9;
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
  font-weight: 700;
  font-size: 10px;
  padding: 2px 10px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);

  &:hover {
    background: rgba(255, 255, 255, 0.35);
  }
}

/* 更多弹窗 */
.fc .fc-more-popover {
  background: rgba(255, 255, 255, 1);
  border: 1px solid rgba(255, 255, 255, 0.35);
  border-radius: 16px;
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  animation: morePopoverIn 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);

  .fc-popover-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px 14px;
    background: rgba(255, 255, 255, 0.15);
    border-bottom: 1px solid rgba(255, 255, 255, 0.2);

    .fc-popover-title {
      color: black;
      font-size: 13px;
      font-weight: 700;
      letter-spacing: 0.5px;
    }

    .fc-popover-close {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 20px;
      height: 20px;
      border-radius: 50%;
      color: black;
      background: rgba(255, 255, 255, 0.15);
      font-size: 14px;
      line-height: 1;
      transition: all 0.2s ease;

      &:hover {
        background: #ff6b6b;
        color: white;
        transform: rotate(90deg);
      }
    }
  }

  .fc-popover-body {
    padding: 0 10px 10px;
    min-width: 220px;
    max-height: 300px;
    overflow-y: auto;

    .fc-more-popover-misc {
      display: none;
    }

    .fc-daygrid-event {
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
      margin-bottom: 6px;

      &:last-child {
        margin-bottom: 0;
      }
    }
  }
}

@keyframes morePopoverIn {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(-8px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
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
  cursor: pointer;
  opacity: 0;
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;

  svg {
    display: block;
    flex-shrink: 0;
  }

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
  background: rgba(255, 255, 255, 0.28);
  border: 1px solid rgba(255, 255, 255, 0.35);
  border-radius: 10px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.25);
  padding: 4px;
  z-index: 1000;
  min-width: 120px;
  animation: contextMenuIn 0.15s cubic-bezier(0.34, 1.56, 0.64, 1);
  transform-origin: top left;
}

.context-menu-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 7px 12px;
  border-radius: 7px;
  cursor: pointer;
  font-size: 12px;
  color: white;
  transition: all 0.2s ease;

  svg {
    flex-shrink: 0;
  }

  &:hover {
    background: rgba(255, 255, 255, 0.18);
  }
}

@keyframes contextMenuIn {
  from {
    opacity: 0;
    transform: scale(0.92) translate(-4px, -4px);
  }
  to {
    opacity: 1;
    transform: scale(1) translate(0, 0);
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
  z-index: 2000000;
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
      border-color: #ff5f8f;
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

.blur-slider {
  width: 100%;
  accent-color: #ff5f8f;
  cursor: pointer;
}

.wallpaper-row {
  display: flex;
  align-items: center;
  gap: 12px;
}

.wallpaper-preview {
  width: 120px;
  height: 68px;
  border-radius: 10px;
  background-size: cover;
  background-position: center;
  border: 1px solid rgba(255, 255, 255, 0.35);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.08);
  flex-shrink: 0;
}

.wallpaper-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.btn-mini {
  padding: 6px 14px;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  color: white;
  background: rgba(255, 95, 143, 0.28);
  border: 1px solid rgba(255, 255, 255, 0.25);
  cursor: pointer;
  transition: all 0.2s ease;

  &:hover:not(:disabled) {
    background: rgba(255, 95, 143, 0.5);
  }

  &:disabled {
    opacity: 0.5;
    cursor: default;
  }
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
</style>
