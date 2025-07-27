<script setup lang="ts">
import ThreeScene from './components/ThreeScene.vue'
</script>

<template>
  <div class="app">
    <div class="header">
      <h1>QuTu - 3D模型查看器</h1>
      <p>拖拽鼠标旋转模型，松开后自动回到正视图并旋转</p>
    </div>
    <ThreeScene />
  </div>
</template>

<style scoped>
.app {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.header {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 100;
  background: rgba(255, 255, 255, 0.9);
  padding: 15px 20px;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.header h1 {
  margin: 0 0 8px 0;
  font-size: 24px;
  color: #333;
  font-weight: 600;
}

.header p {
  margin: 0;
  font-size: 14px;
  color: #666;
  line-height: 1.4;
}
</style>

<style>
:root {
  font-family: Inter, Avenir, Helvetica, Arial, sans-serif;
  font-size: 16px;
  line-height: 24px;
  font-weight: 400;
  color: #0f0f0f;
  background-color: #f6f6f6;
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

body {
  margin: 0;
  overflow: hidden;
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #f6f6f6;
    background-color: #2f2f2f;
  }
  
  .header {
    background: rgba(47, 47, 47, 0.9);
  }
  
  .header h1 {
    color: #f6f6f6;
  }
  
  .header p {
    color: #ccc;
  }
}
</style>