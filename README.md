# CalendarNote

操作便捷的日历便笺，用于快速记录以及查看工作安排。

## 功能特性

- 日历视图：直观查看每月日期，点击日期查看/添加便签
- 便签管理：支持创建、查看、删除便签
- 颜色标记：8种颜色可选，方便分类管理
- 本地存储：支持Tauri后端存储或浏览器本地存储
- 响应式设计：自适应窗口大小

## 技术栈

- 前端：Vue 3 + TypeScript + Element Plus
- 后端：Tauri 2.x (Rust)
- 构建工具：Vite

## 开发

```bash
# 安装依赖
npm install

# 开发模式
npm run dev

# 构建
npm run build

# Tauri 开发模式
npm run tauri dev

# Tauri 构建
npm run tauri build
```

## 项目结构

```
CalendarNote/
├── src/                    # 前端源码
│   ├── components/        # Vue组件
│   │   ├── CalendarView.vue  # 日历视图
│   │   ├── NoteCard.vue     # 便签卡片
│   │   └── NoteForm.vue     # 便签表单
│   ├── App.vue            # 主组件
│   └── main.ts            # 入口文件
├── src-tauri/             # Tauri后端
│   ├── src/
│   │   ├── lib.rs         # Rust库
│   │   └── main.rs        # 入口
│   └── tauri.conf.json    # Tauri配置
├── package.json
└── vite.config.ts
```

## 许可证

MIT
