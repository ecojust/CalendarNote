<template>
  <div id="app">
    <div class="app-container">
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
import { ref, onMounted } from "vue";
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

onMounted(async () => {
  await loadNotes();
});

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
  overflow: hidden;
  background: transparent;
}

.app-container {
  width: 100%;
  height: 100%;
  background: rgba(255, 245, 250, 0);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(219, 112, 147, 0.15);
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #5a4a6a;
    background-color: transparent;
  }

  .app-container {
    background: rgba(60, 40, 60, 0);
  }
}
</style>
