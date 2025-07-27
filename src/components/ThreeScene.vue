<template>
  <div ref="container" class="three-container">
    <div class="controls">
      <input 
        ref="fileInput" 
        type="file" 
        accept=".glb,.gltf" 
        @change="loadGLBModel" 
        style="display: none"
      />
      <button @click="openFileDialog" class="load-button">
        📁 加载GLB模型
      </button>
      <button @click="resetToDefaultCube" class="reset-button">
        🔄 重置为方块
      </button>
      <div class="light-control">
        <label for="lightIntensity">💡 光照强度</label>
        <input 
          id="lightIntensity"
          type="range" 
          min="0.1" 
          max="3" 
          step="0.1" 
          v-model="lightIntensity" 
          @input="updateLightIntensity"
          class="light-slider"
        />
        <span class="light-value">{{ lightIntensity }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'

const container = ref<HTMLDivElement>()
const fileInput = ref<HTMLInputElement>()

let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let currentModel: THREE.Object3D | null = null
let animationId: number
let gltfLoader: GLTFLoader
let directionalLight: THREE.DirectionalLight
let ambientLight: THREE.AmbientLight

// 响应式变量
const lightIntensity = ref(1.0)

// 鼠标控制相关变量
let isMouseDown = false
let mouseX = 0
let mouseY = 0
let targetRotationX = 0
let targetRotationY = 0
let currentRotationX = 0
let currentRotationY = 0

// 自动旋转相关变量
let autoRotate = true
let autoRotateSpeed = 0.01

// 初始化3D场景
function initThree() {
  if (!container.value) return

  // 创建场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0xf0f0f0)

  // 创建相机
  camera = new THREE.PerspectiveCamera(
    75,
    container.value.clientWidth / container.value.clientHeight,
    0.1,
    1000
  )
  camera.position.z = 5

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  container.value.appendChild(renderer.domElement)

  // 初始化GLTFLoader
  gltfLoader = new GLTFLoader()

  // 创建默认方块模型
  createDefaultCube()

  // 添加光源
  ambientLight = new THREE.AmbientLight(0x404040, 0.6)
  scene.add(ambientLight)

  directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
  directionalLight.position.set(10, 10, 5)
  directionalLight.castShadow = true
  scene.add(directionalLight)

  // 添加地面（用于接收阴影）
  const planeGeometry = new THREE.PlaneGeometry(10, 10)
  const planeMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  plane.position.y = -2
  plane.receiveShadow = true
  scene.add(plane)

  // 添加鼠标事件监听
  addEventListeners()

  // 开始渲染循环
  animate()
}

// 添加事件监听器
function addEventListeners() {
  if (!container.value) return

  const canvas = renderer.domElement

  canvas.addEventListener('mousedown', onMouseDown)
  canvas.addEventListener('mousemove', onMouseMove)
  canvas.addEventListener('mouseup', onMouseUp)
  canvas.addEventListener('mouseleave', onMouseUp)
  canvas.addEventListener('wheel', onMouseWheel)

  // 防止右键菜单
  canvas.addEventListener('contextmenu', (e) => e.preventDefault())
}

// 鼠标按下事件
function onMouseDown(event: MouseEvent) {
  if (event.button === 2) { // 右键
    resetCamera()
    return
  }
  
  if (event.button === 0) { // 左键
    isMouseDown = true
    autoRotate = false
    mouseX = event.clientX
    mouseY = event.clientY
  }
}

// 鼠标移动事件
function onMouseMove(event: MouseEvent) {
  if (!isMouseDown) return

  const deltaX = event.clientX - mouseX
  const deltaY = event.clientY - mouseY

  targetRotationY += deltaX * 0.01
  targetRotationX += deltaY * 0.01

  // 限制X轴旋转角度
  targetRotationX = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, targetRotationX))

  mouseX = event.clientX
  mouseY = event.clientY
}

// 鼠标释放事件
function onMouseUp() {
  if (isMouseDown) {
    isMouseDown = false
    // 延迟恢复自动旋转和回到正视图
    setTimeout(() => {
      autoRotate = true
      // 平滑回到正视图
      targetRotationX = 0
      targetRotationY = 0
    }, 1000)
  }
}

// 动画循环
function animate() {
  animationId = requestAnimationFrame(animate)

  // 自动旋转
  if (autoRotate && !isMouseDown) {
    targetRotationY += autoRotateSpeed
  }

  // 平滑插值到目标旋转角度
  currentRotationX += (targetRotationX - currentRotationX) * 0.1
  currentRotationY += (targetRotationY - currentRotationY) * 0.1

  // 应用旋转到当前模型
  if (currentModel) {
    currentModel.rotation.x = currentRotationX
    currentModel.rotation.y = currentRotationY
  }

  renderer.render(scene, camera)
}

// 创建默认方块模型
function createDefaultCube() {
  // 移除当前模型
  if (currentModel) {
    scene.remove(currentModel)
  }

  // 创建方块几何体
  const geometry = new THREE.BoxGeometry(2, 2, 2)
  const material = new THREE.MeshLambertMaterial({
    color: 0x4a90e2,
    transparent: true,
    opacity: 0.8
  })
  const cube = new THREE.Mesh(geometry, material)
  cube.castShadow = true

  // 添加边框线
  const edges = new THREE.EdgesGeometry(geometry)
  const lineMaterial = new THREE.LineBasicMaterial({ color: 0x000000, linewidth: 2 })
  const wireframe = new THREE.LineSegments(edges, lineMaterial)
  cube.add(wireframe)

  currentModel = cube
  scene.add(currentModel)
}

// 打开文件选择对话框
function openFileDialog() {
  fileInput.value?.click()
}

// 加载GLB模型
function loadGLBModel(event: Event) {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  
  if (!file) return

  const url = URL.createObjectURL(file)
  
  gltfLoader.load(
    url,
    (gltf) => {
      // 移除当前模型
      if (currentModel) {
        scene.remove(currentModel)
      }

      // 添加新模型
      currentModel = gltf.scene
      
      // 设置模型属性
      currentModel.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.castShadow = true
          child.receiveShadow = true
        }
      })

      // 计算模型边界盒，调整大小和位置
      const box = new THREE.Box3().setFromObject(currentModel)
      const size = box.getSize(new THREE.Vector3())
      const maxDim = Math.max(size.x, size.y, size.z)
      const scale = 2 / maxDim // 缩放到合适大小
      currentModel.scale.setScalar(scale)

      // 居中模型
      const center = box.getCenter(new THREE.Vector3())
      currentModel.position.sub(center.multiplyScalar(scale))

      scene.add(currentModel)
      
      // 重置旋转
      targetRotationX = 0
      targetRotationY = 0
      currentRotationX = 0
      currentRotationY = 0
      
      // 清理URL对象
      URL.revokeObjectURL(url)
    },
    (progress) => {
      console.log('加载进度:', (progress.loaded / progress.total * 100) + '%')
    },
    (error) => {
      console.error('加载GLB模型失败:', error)
      alert('加载模型失败，请检查文件格式是否正确')
      URL.revokeObjectURL(url)
    }
  )
  
  // 清空文件输入
  target.value = ''
}

// 重置为默认方块
function resetToDefaultCube() {
  createDefaultCube()
  // 重置旋转
  targetRotationX = 0
  targetRotationY = 0
  currentRotationX = 0
  currentRotationY = 0
}

// 鼠标滚轮事件
function onMouseWheel(event: WheelEvent) {
  event.preventDefault()
  
  const delta = event.deltaY > 0 ? 1.1 : 0.9
  camera.position.multiplyScalar(delta)
  
  // 限制相机距离
  const distance = camera.position.length()
  if (distance < 2) {
    camera.position.normalize().multiplyScalar(2)
  } else if (distance > 20) {
    camera.position.normalize().multiplyScalar(20)
  }
}

// 重置相机位置
function resetCamera() {
  camera.position.set(0, 0, 5)
  camera.lookAt(0, 0, 0)
}

// 更新光照强度
function updateLightIntensity() {
  if (directionalLight && ambientLight) {
    directionalLight.intensity = lightIntensity.value * 0.8
    ambientLight.intensity = lightIntensity.value * 0.6
  }
}

// 处理窗口大小变化
function onWindowResize() {
  if (!container.value) return

  camera.aspect = container.value.clientWidth / container.value.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
}

// 组件挂载时初始化
onMounted(() => {
  initThree()
  window.addEventListener('resize', onWindowResize)
})

// 组件卸载时清理
onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', onWindowResize)
  if (container.value && renderer) {
    container.value.removeChild(renderer.domElement)
  }
})
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  cursor: grab;
  position: relative;
}

.three-container:active {
  cursor: grabbing;
}

.controls {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 100;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.load-button,
.reset-button {
  padding: 12px 16px;
  border: none;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  min-width: 140px;
  text-align: left;
}

.load-button:hover,
.reset-button:hover {
  background: rgba(255, 255, 255, 1);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.load-button:active,
.reset-button:active {
  transform: translateY(0);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.light-control {
  background: rgba(255, 255, 255, 0.9);
  padding: 12px 16px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  min-width: 140px;
}

.light-control label {
  display: block;
  font-size: 14px;
  font-weight: 500;
  color: #333;
  margin-bottom: 8px;
}

.light-slider {
  width: 100%;
  height: 4px;
  border-radius: 2px;
  background: #ddd;
  outline: none;
  margin-bottom: 8px;
  cursor: pointer;
}

.light-slider::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #4a90e2;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.light-slider::-moz-range-thumb {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #4a90e2;
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.light-value {
  font-size: 12px;
  color: #666;
  font-weight: 500;
}

@media (prefers-color-scheme: dark) {
  .load-button,
  .reset-button {
    background: rgba(47, 47, 47, 0.9);
    color: #f6f6f6;
  }
  
  .load-button:hover,
  .reset-button:hover {
    background: rgba(47, 47, 47, 1);
  }
  
  .light-control {
    background: rgba(47, 47, 47, 0.9);
  }
  
  .light-control label {
    color: #f6f6f6;
  }
  
  .light-slider {
    background: #555;
  }
  
  .light-value {
    color: #ccc;
  }
}
</style>