<template>
  <div id="app">
    <div
      class="app-container"
      :style="{
        '--pointer-x': `${pointer.x}%`,
        '--pointer-y': `${pointer.y}%`,
      }"
      @pointerleave="resetPointer"
    >
      <div class="glass-glow" aria-hidden="true" />
      <CalendarView
        :notes="notes"
        :config="config"
        @add-note="handleAddNote"
        @delete-note="handleDeleteNote"
        @move-note="handleMoveNote"
        @update-note="handleUpdateNote"
        @month-change="handleMonthChange"
        @clear-month="handleClearMonth"
        @update-config="handleConfigChange"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import CalendarView from "./components/CalendarView.vue";
// @ts-ignore
import Database from "@tauri-apps/plugin-sql";

interface Note {
  id: string;
  title: string;
  content: string;
  date: string;
  color: string;
  reminder?: string;
  created_at: string;
}

const notes = ref<Note[]>([]);
const config = ref<Record<string, string>>({});
const pointer = ref({ x: 50, y: 50 });
let animationFrameId: number | null = null;
let db: any = null;

async function getDb() {
  if (!db) {
    db = await Database.load("sqlite:calendar.db");
  }
  return db;
}

onMounted(async () => {
  window.addEventListener("pointermove", handlePointerMove, { passive: true });
  await loadConfig();
});

onBeforeUnmount(() => {
  window.removeEventListener("pointermove", handlePointerMove);
  if (animationFrameId !== null) {
    cancelAnimationFrame(animationFrameId);
  }
});

function handlePointerMove(event: PointerEvent) {
  if (animationFrameId !== null) {
    return;
  }

  animationFrameId = requestAnimationFrame(() => {
    pointer.value = {
      x: (event.clientX / window.innerWidth) * 100,
      y: (event.clientY / window.innerHeight) * 100,
    };
    animationFrameId = null;
  });
}

function resetPointer() {
  pointer.value = { x: 50, y: 50 };
}

async function loadNotes(start?: string, end?: string) {
  if (!start || !end) {
    const now = new Date();
    const first = new Date(now.getFullYear(), now.getMonth(), 1);
    const next = new Date(now.getFullYear(), now.getMonth() + 1, 1);
    start = formatYmd(first);
    end = formatYmd(next);
  }
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      notes.value = await db.select(
        "SELECT * FROM notes WHERE date >= $1 AND date < $2 ORDER BY created_at DESC",
        [start, end],
      );
    } else {
      const stored = localStorage.getItem("calendar-notes");
      const all = stored ? JSON.parse(stored) : [];
      notes.value = all.filter(
        (n: Note) => n.date >= (start as string) && n.date < (end as string),
      );
    }
  } catch (e) {
    console.error("Failed to load notes:", e);
  }
}

async function loadConfig() {
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      const rows = (await db.select("SELECT key, value FROM config")) as {
        key: string;
        value: string;
      }[];
      config.value = Object.fromEntries(rows.map((r) => [r.key, r.value]));
    } else {
      config.value = {
        "bg-blur": localStorage.getItem("calendar-bg-blur") ?? "20",
      };
    }
  } catch (e) {
    console.error("Failed to load config:", e);
  }
}

async function handleConfigChange(key: string, value: string) {
  config.value = { ...config.value, [key]: value };
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      await db.execute(
        "INSERT INTO config (key, value) VALUES ($1, $2) ON CONFLICT(key) DO UPDATE SET value = excluded.value",
        [key, value],
      );
    } else {
      localStorage.setItem("calendar-" + key, value);
    }
  } catch (e) {
    console.error("Failed to save config:", e);
  }
}

async function handleUpdateNote(id: string, patch: Partial<Note>) {
  const entries = Object.entries(patch);
  if (entries.length === 0) return;
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      const setClause = entries
        .map(([key], i) => `${key} = $${i + 1}`)
        .join(", ");
      const params = [...entries.map(([, v]) => v || null), id];
      await db.execute(
        `UPDATE notes SET ${setClause} WHERE id = $${entries.length + 1}`,
        params,
      );
    } else {
      localStorage.setItem(
        "calendar-notes",
        JSON.stringify(
          notes.value.map((n) => (n.id === id ? { ...n, ...patch } : n)),
        ),
      );
    }
    notes.value = notes.value.map((n) =>
      n.id === id ? { ...n, ...patch } : n,
    );
  } catch (e) {
    console.error("Failed to update note:", e);
  }
}

function handleMonthChange(start: string, end: string) {
  loadNotes(start, end);
}

async function handleClearMonth(start: string, end: string) {
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      await db.execute("DELETE FROM notes WHERE date >= $1 AND date < $2", [
        start,
        end,
      ]);
    } else {
      localStorage.setItem(
        "calendar-notes",
        JSON.stringify(
          notes.value.filter((n) => n.date < start || n.date >= end),
        ),
      );
    }
    notes.value = notes.value.filter((n) => n.date < start || n.date >= end);
  } catch (e) {
    console.error("Failed to clear month:", e);
  }
}

function formatYmd(date: Date) {
  const y = date.getFullYear();
  const m = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");
  return `${y}-${m}-${day}`;
}

async function handleAddNote(note: Note) {
  notes.value = [note, ...notes.value];
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      await db.execute(
        "INSERT INTO notes (id, title, content, date, color, reminder, created_at) VALUES ($1, $2, $3, $4, $5, $6, $7)",
        [
          note.id,
          note.title,
          note.content,
          note.date,
          note.color,
          note.reminder || null,
          note.created_at,
        ],
      );
    } else {
      localStorage.setItem("calendar-notes", JSON.stringify(notes.value));
    }
  } catch (e) {
    console.error("Failed to add note:", e);
  }
}

async function handleDeleteNote(id: string) {
  notes.value = notes.value.filter((n) => n.id !== id);
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await getDb();
      await db.execute("DELETE FROM notes WHERE id = $1", [id]);
    } else {
      localStorage.setItem("calendar-notes", JSON.stringify(notes.value));
    }
  } catch (e) {
    console.error("Failed to delete note:", e);
  }
}

async function handleMoveNote(id: string, newDate: string) {
  console.log("[App] handleMoveNote called:", id, "->", newDate);
  const noteIndex = notes.value.findIndex((n) => n.id === id);
  console.log("[App] noteIndex:", noteIndex);
  if (noteIndex !== -1) {
    const oldDate = notes.value[noteIndex].date;
    console.log("[App] oldDate:", oldDate, "-> newDate:", newDate);
    notes.value[noteIndex] = {
      ...notes.value[noteIndex],
      date: newDate,
    };
    notes.value = [...notes.value];
    console.log("[App] notes.value updated, length:", notes.value.length);

    try {
      // @ts-ignore
      if (window.__TAURI__) {
        const db = await getDb();
        await db.execute("UPDATE notes SET date = $1 WHERE id = $2", [
          newDate,
          id,
        ]);
      } else {
        localStorage.setItem("calendar-notes", JSON.stringify(notes.value));
      }
    } catch (e) {
      console.error("Failed to move note:", e);
    }
  }
}
</script>

<style lang="less">
:root {
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue",
    Arial, sans-serif;
  font-size: 16px;
  line-height: 24px;
  font-weight: 400;
  color: #5a4a6a;
  background-color: transparent;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-text-size-adjust: 100%;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  width: 100vw;
  height: 100vh;
  border-radius: 10px;
  overflow: hidden;
  background: transparent;
}

.app-container {
  position: relative;
  width: 100%;
  height: 100%;
  background:
    linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.48),
      rgba(255, 255, 255, 0.1)
    ),
    radial-gradient(
      circle at var(--pointer-x) var(--pointer-y),
      rgba(255, 255, 255, 0.82) 0%,
      rgba(255, 255, 255, 0.3) 16%,
      rgba(255, 255, 255, 0.12) 26%,
      rgba(255, 255, 255, 0) 46%
    );
  backdrop-filter: blur(0px) saturate(150%) brightness(1.12);
  -webkit-backdrop-filter: blur(0px) saturate(150%) brightness(1.12);
  border-radius: 10px;
  overflow: hidden;
  border: 0px solid rgba(255, 255, 255, 0);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.14),
    inset 0 -1px 0 rgba(255, 255, 255, 0.05),
    0 18px 40px rgba(176, 126, 184, 0.12),
    0 8px 22px rgba(124, 98, 150, 0.1);
  transition:
    background 220ms ease-out,
    box-shadow 220ms ease-out,
    border-color 220ms ease-out;
  isolation: isolate;
}

.glass-glow {
  position: absolute;
  inset: -18% -12%;
  border-radius: 10px;
  border: none;
  pointer-events: none;
  background: radial-gradient(
    circle at var(--pointer-x) var(--pointer-y),
    rgba(255, 255, 255, 0.7) 0%,
    rgba(255, 255, 255, 0.18) 18%,
    rgba(255, 255, 255, 0) 42%
  );
  filter: blur(22px);
  opacity: 0.96;
  z-index: 0;
}

.app-container > * {
  position: relative;
  z-index: 1;
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #5a4a6a;
    background-color: transparent;
  }

  .app-container {
    background:
      linear-gradient(135deg, rgba(72, 54, 86, 0.42), rgba(60, 40, 60, 0.08)),
      radial-gradient(
        circle at var(--pointer-x) var(--pointer-y),
        rgba(255, 255, 255, 0.38) 0%,
        rgba(255, 255, 255, 0.13) 18%,
        rgba(255, 255, 255, 0.05) 30%,
        rgba(255, 255, 255, 0) 52%
      );
    border-color: rgba(255, 255, 255, 0.08);
    box-shadow:
      inset 0 1px 0 rgba(255, 255, 255, 0.08),
      inset 0 -1px 0 rgba(255, 255, 255, 0.04),
      0 18px 40px rgba(24, 18, 36, 0.18),
      0 8px 22px rgba(70, 58, 84, 0.12);
  }
}
</style>
