<template>
  <div class="note-card" :style="{ borderLeftColor: note.color }">
    <div class="note-content">{{ note.content }}</div>
    <div class="note-footer">
      <span class="note-time">{{ formatTime }}</span>
      <button class="delete-btn" @click="$emit('delete')">
        <svg width="14" height="14" viewBox="0 0 14 14">
          <path d="M3 4H11M5 4V3C5 2.45 5.45 2 6 2H8C8.55 2 9 2.45 9 3V4M4 4V11C4 11.55 4.45 12 5 12H9C9.55 12 10 11.55 10 11V4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";

interface Note {
  id: string;
  content: string;
  date: string;
  color: string;
  created_at: string;
}

const props = defineProps<{
  note: Note;
}>();

defineEmits<{
  (e: "delete"): void;
}>();

const formatTime = computed(() => {
  const date = new Date(props.note.created_at);
  return `${date.getHours().toString().padStart(2, "0")}:${date.getMinutes().toString().padStart(2, "0")}`;
});
</script>

<style lang="less" scoped>
.note-card {
  background: linear-gradient(135deg, #fff5f8 0%, #ffffff 100%);
  border-radius: 16px;
  padding: 14px 16px;
  border-left: 4px solid;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 3px 10px rgba(214, 51, 132, 0.08);
  
  &:hover {
    background: linear-gradient(135deg, #ffeef3 0%, #fff5f8 100%);
    transform: translateX(6px) scale(1.01);
    box-shadow: 0 6px 20px rgba(214, 51, 132, 0.15);
  }
}

.note-content {
  color: #5a4a6a;
  font-size: 13px;
  line-height: 1.5;
  margin-bottom: 10px;
  white-space: pre-wrap;
  word-break: break-word;
}

.note-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.note-time {
  color: #c9a0c9;
  font-size: 10px;
  font-weight: 500;
}

.delete-btn {
  background: none;
  border: none;
  color: #e0b0e0;
  cursor: pointer;
  padding: 4px;
  border-radius: 8px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  justify-content: center;
  
  &:hover {
    background: #ffeef3;
    color: #ff6b9d;
    transform: scale(1.15) rotate(5deg);
  }
}
</style>
