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
        @add-note="handleAddNote"
        @delete-note="handleDeleteNote"
        @move-note="handleMoveNote"
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
const pointer = ref({ x: 50, y: 50 });
let animationFrameId: number | null = null;

onMounted(async () => {
  await loadNotes();
  window.addEventListener("pointermove", handlePointerMove, { passive: true });
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

async function loadNotes() {
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await Database.load("sqlite:calendar.db");
      notes.value = await db.select(
        "SELECT * FROM notes ORDER BY created_at DESC",
      );
      await db.close();
    } else {
      const stored = localStorage.getItem("calendar-notes");
      if (stored) {
        notes.value = JSON.parse(stored);
      }
    }
  } catch (e) {
    console.error("Failed to load notes:", e);
  }
}

async function handleAddNote(note: Note) {
  notes.value.unshift(note);
  try {
    // @ts-ignore
    if (window.__TAURI__) {
      const db = await Database.load("sqlite:calendar.db");
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
      await db.close();
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
      const db = await Database.load("sqlite:calendar.db");
      await db.execute("DELETE FROM notes WHERE id = $1", [id]);
      await db.close();
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
        const db = await Database.load("sqlite:calendar.db");
        await db.execute("UPDATE notes SET date = $1 WHERE id = $2", [
          newDate,
          id,
        ]);
        await db.close();
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
  border-radius: 20px;

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
  backdrop-filter: blur(24px) saturate(150%) brightness(1.12);
  -webkit-backdrop-filter: blur(24px) saturate(150%) brightness(1.12);
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.45);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.55),
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
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 1);
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
    border-color: rgba(255, 255, 255, 0.12);
    box-shadow:
      inset 0 1px 0 rgba(255, 255, 255, 0.2),
      0 18px 40px rgba(24, 18, 36, 0.18),
      0 8px 22px rgba(70, 58, 84, 0.12);
  }
}
</style>
