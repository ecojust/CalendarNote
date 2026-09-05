<template>
  <div class="note-form">
    <el-input
      v-model="content"
      type="textarea"
      :rows="4"
      placeholder="输入便签内容..."
      resize="none"
    />
    
    <div class="color-picker">
      <span class="label">颜色：</span>
      <div class="colors">
        <span
          v-for="color in colors"
          :key="color"
          class="color-option"
          :class="{ active: selectedColor === color }"
          :style="{ backgroundColor: color }"
          @click="selectedColor = color"
        ></span>
      </div>
    </div>
    
    <div class="form-actions">
      <el-button @click="$emit('cancel')">取消</el-button>
      <el-button type="primary" @click="handleSubmit" :disabled="!content.trim()">
        确定
      </el-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

interface Note {
  id: string;
  content: string;
  date: string;
  color: string;
  created_at: string;
}

const props = defineProps<{
  date: string;
}>();

const emit = defineEmits<{
  (e: "submit", note: Note): void;
  (e: "cancel"): void;
}>();

const content = ref("");
const selectedColor = ref("#ff5f8f");

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

function handleSubmit() {
  if (!content.value.trim()) return;
  
  const note: Note = {
    id: Date.now().toString() + Math.random().toString(36).substr(2, 9),
    content: content.value.trim(),
    date: props.date,
    color: selectedColor.value,
    created_at: new Date().toISOString(),
  };
  
  emit("submit", note);
  content.value = "";
}
</script>

<style lang="less" scoped>
.note-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.color-picker {
  display: flex;
  align-items: center;
  gap: 12px;
  
  .label {
    color: #d63384;
    font-size: 13px;
    font-weight: 600;
  }
}

.colors {
  display: flex;
  gap: 8px;
}

.color-option {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 3px solid transparent;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  
  &:hover {
    transform: scale(1.2);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  }
  
  &.active {
    border-color: white;
    transform: scale(1.25);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.25);
  }
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 8px;
}
</style>
