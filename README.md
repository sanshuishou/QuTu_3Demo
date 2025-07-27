# QuTu - 3D模型查看器

一个基于 Tauri2 + Vue3 + TypeScript + Three.js 构建的现代化3D模型查看器应用。

## ✨ 功能特性

### 🎮 交互控制
- **鼠标拖拽旋转** - 左键按住拖拽可自由旋转3D模型
- **滚轮缩放** - 鼠标滚轮控制视角远近（距离限制：2-20单位）
- **右键重置** - 右键点击立即重置相机到默认位置
- **自动回转** - 松开鼠标后自动回到正视图并持续旋转

### 📁 模型支持
- **GLB/GLTF加载** - 支持加载标准的GLB和GLTF格式3D模型
- **智能缩放** - 自动调整模型大小和位置以适配视野
- **阴影效果** - 自动为加载的模型启用阴影渲染
- **默认方块** - 内置蓝色半透明方块作为默认展示模型

### 💡 光照系统
- **实时调节** - 光照强度滑条（0.1-3.0范围）
- **环境光+方向光** - 组合光照提供最佳视觉效果
- **阴影投射** - 支持实时阴影计算和渲染

### 🎨 界面设计
- **现代化UI** - 半透明控制面板，支持深色模式
- **响应式布局** - 自适应窗口大小变化
- **流畅动画** - 平滑的旋转和缩放过渡效果

## 🛠️ 技术栈

- **前端框架**: Vue 3 + TypeScript
- **3D渲染**: Three.js
- **桌面应用**: Tauri 2
- **构建工具**: Vite
- **开发语言**: TypeScript + Rust

## 🚀 快速开始

### 环境要求
- Node.js 16+
- Rust 1.70+
- 支持的操作系统：Windows、macOS、Linux

### 安装依赖
```bash
npm install
```

### 开发模式
```bash
npm run dev
```

### 构建应用
```bash
npm run build
```

### 运行Tauri应用
```bash
npm run tauri dev
```

## 📖 使用说明

1. **加载模型**: 点击右上角"📁 加载GLB模型"按钮选择GLB/GLTF文件
2. **旋转模型**: 左键按住拖拽进行模型旋转
3. **缩放视角**: 使用鼠标滚轮调整观察距离
4. **重置视角**: 右键点击场景重置到默认视角
5. **调节光照**: 拖动光照强度滑条调整场景亮度
6. **重置模型**: 点击"🔄 重置为方块"回到默认状态

## 🔧 开发环境配置

### 推荐IDE
- [VS Code](https://code.visualstudio.com/)
- [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) - Vue 3支持
- [Tauri](https://marketplace.visualstudio.com/items?itemName=tauri-apps.tauri-vscode) - Tauri开发支持
- [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) - Rust语言支持

### TypeScript支持
项目使用Vue 3的`<script setup>`语法，完整的TypeScript类型支持。如需在`.vue`文件中获得完整的类型检查，可启用Volar的Take Over模式。

## 📁 项目结构
```
QuTu/
├── src/                    # Vue前端源码
│   ├── components/         # Vue组件
│   │   └── ThreeScene.vue # 3D场景核心组件
│   ├── App.vue            # 主应用组件
│   └── main.ts            # 应用入口
├── src-tauri/             # Tauri后端源码
│   ├── src/               # Rust源码
│   └── tauri.conf.json    # Tauri配置
├── public/                # 静态资源
└── package.json           # 项目依赖
```

## 🤝 贡献指南

欢迎提交Issue和Pull Request来改进这个项目！

## 📄 许可证

本项目采用 MIT 许可证。
