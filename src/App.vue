<template>
  <div class="app">
    <header class="header">
      <div class="header-inner">
        <div class="header-left">
          <h1 class="header-title">AI智能修图 V2.5</h1>
          <span class="header-sub">—— 马赛克消除</span>
        </div>
        <div class="badges">
          <span class="badge">🟢 GPU 加速</span>
          <span class="badge">🔥 深度学习</span>
          <span class="badge">⚡ 实时推理</span>
        </div>
      </div>
    </header>

    <main class="main">
      <!-- 上传区域 -->
      <div v-if="!uploadedImage && !isProcessing && !isProcessed" class="upload-zone" @click="triggerUpload" @dragover.prevent @drop.prevent="handleDrop">
        <div class="upload-inner">
          <div class="upload-icon">📁</div>
          <h2>上传图片</h2>
          <p>点击或拖拽图片到此区域</p>
          <p class="upload-hint">支持 JPG / PNG / WebP 格式</p>
        </div>
        <input ref="fileInput" type="file" accept="image/*" @change="handleUpload" hidden />
      </div>

      <!-- 图片展示 -->
      <div v-if="uploadedImage" class="workspace">
        <div class="image-container">
          <div class="image-label">
            <span>{{ isProcessed ? '✨ AI 还原结果' : '📷 原始马赛克图片' }}</span>
          </div>
          <div class="image-wrapper" :class="{ 'processing': isProcessing }">
            <!-- 底层：原图 -->
            <img class="img-base" :src="uploadedImage" alt="原图" />
            <!-- 上层：结果图，从上到下覆盖 -->
            <img
              v-if="isProcessed || isTransitioning"
              class="img-result"
              :class="{ 'img-reveal': isTransitioning, 'img-done': isRevealed }"
              :src="resultImage"
              alt="还原结果"
            />
            <!-- 处理中的扫描线动画 -->
            <div v-if="isProcessing" class="scan-line"></div>
            <!-- 处理中的遮罩 -->
            <div v-if="isProcessing" class="processing-overlay">
              <div class="spinner"></div>
            </div>
          </div>
        </div>

        <!-- 进度信息 -->
        <div v-if="isProcessing" class="progress-section">
          <div class="progress-bar-container">
            <div class="progress-bar" :style="{ width: progress + '%' }"></div>
          </div>
          <div class="progress-text">{{ progressText }}</div>
          <div class="tech-log">{{ techLog }}</div>
        </div>
      </div>

      <!-- 操作按钮（始终显示） -->
      <div class="actions">
        <button
          v-if="!isProcessing && !isProcessed"
          class="btn-primary btn-glow"
          @click="uploadedImage ? startProcess() : triggerUpload()"
        >
          🚀 一键去除马赛克
        </button>
        <button
          v-if="isProcessing"
          class="btn-primary"
          disabled
        >
          ⏳ AI 正在努力还原中...
        </button>
      </div>

      <!-- 完成提示 -->
      <div v-if="isProcessed" class="result-info">
        <span class="success-icon">✅</span>
        <span class="success-text">还原成功！</span>
      </div>
    </main>

    <!-- 底部 -->
    <footer class="footer">
      <p>Powered by MosaicAI™ Deep Learning Engine | 算力支持: NVIDIA A100 × 8 | 专利号: ZL2024XXXXXXXX.X</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const fileInput = ref(null)
const uploadedImage = ref('')
const isProcessing = ref(false)
const isProcessed = ref(false)
const isTransitioning = ref(false)
const isRevealed = ref(false)
const progress = ref(0)
const progressStep = ref(0)
const resultImage = ref('/gege.jpg')

const progressSteps = [
  { text: '🧠 识别马赛克区域...', log: '' },
  { text: '🔬 像素级深度推理中...', log: '' },
  { text: '🎨 纹理细节还原中...', log: '' },
  { text: '✨ 最终优化处理...', log: '' },
]

const progressText = computed(() => {
  if (progressStep.value < progressSteps.length) {
    return progressSteps[progressStep.value].text
  }
  return '✅ 处理完成！'
})

const techLog = computed(() => {
  if (progressStep.value < progressSteps.length) {
    return progressSteps[progressStep.value].log
  }
  return '[INFO] 推理完成, 总耗时 3.42s'
})

function triggerUpload() {
  fileInput.value.click()
}

function handleUpload(e) {
  const file = e.target.files[0]
  if (!file) return
  loadImage(file)
}

function handleDrop(e) {
  const file = e.dataTransfer.files[0]
  if (!file) return
  loadImage(file)
}

function loadImage(file) {
  const reader = new FileReader()
  reader.onload = (e) => {
    uploadedImage.value = e.target.result
    isProcessed.value = false
    isProcessing.value = false
    isRevealed.value = false
    progress.value = 0
    progressStep.value = 0
  }
  reader.readAsDataURL(file)
}

function startProcess() {
  isProcessing.value = true
  progress.value = 0
  progressStep.value = 0

  const totalDuration = 4000 // 4秒假装在处理
  const stepDuration = totalDuration / progressSteps.length
  let elapsed = 0

  const stepTimer = setInterval(() => {
    elapsed += stepDuration
    progressStep.value = Math.min(Math.floor(elapsed / stepDuration), progressSteps.length - 1)
  }, stepDuration)

  const progressTimer = setInterval(() => {
    progress.value += 1
    if (progress.value >= 100) {
      clearInterval(progressTimer)
      clearInterval(stepTimer)
      progressStep.value = progressSteps.length
      setTimeout(() => {
        isProcessing.value = false
        isTransitioning.value = true
        // 过渡动画1.5秒后完成，不移除 img-reveal 类，让 animation forwards 保持最终状态
        setTimeout(() => {
          isTransitioning.value = false
          isRevealed.value = true
          isProcessed.value = true
        }, 1500)
      }, 300)
    }
  }, totalDuration / 100)
}

function reset() {
  uploadedImage.value = ''
  isProcessed.value = false
  isProcessing.value = false
  isRevealed.value = false
  progress.value = 0
  progressStep.value = 0
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Noto Sans SC', -apple-system, BlinkMacSystemFont, sans-serif;
  background: #0a0e1a;
  color: #e0e6f0;
  min-height: 100vh;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header */
.header {
  background: linear-gradient(135deg, #0f1729 0%, #1a1040 100%);
  border-bottom: 1px solid rgba(99, 102, 241, 0.3);
  padding: 28px 24px;
}

.header-inner {
  max-width: 1800px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
}

.header-title {
  font-size: 26px;
  font-weight: 900;
  background: linear-gradient(90deg, #818cf8, #c084fc);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: 1px;
  margin-left: 40px;
}

/* Header Content */
.header-left {
  display: flex;
  align-items: flex-end;
  gap: 12px;
}

.header-sub {
  font-size: 16px;
  font-weight: 600;
  color: #a5b4fc;
  letter-spacing: 4px;
  transform: translateY(4px);
}

.badges {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.badge {
  background: rgba(99, 102, 241, 0.15);
  border: 1px solid rgba(99, 102, 241, 0.3);
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  color: #a5b4fc;
}

/* Main */
.main {
  flex: 1;
  padding: 12px 20px;
  max-width: 900px;
  margin: 0 auto;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

/* Upload Zone */
.upload-zone {
  border: 2px dashed rgba(99, 102, 241, 0.4);
  border-radius: 16px;
  padding: 60px 80px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  background: rgba(99, 102, 241, 0.03);
}

.upload-zone:hover {
  border-color: rgba(99, 102, 241, 0.8);
  background: rgba(99, 102, 241, 0.08);
  transform: translateY(-2px);
}

.upload-inner h2 {
  font-size: 28px;
  margin: 14px 0 8px;
  color: #c7d2fe;
}

.upload-inner p {
  color: #6b7280;
  font-size: 18px;
}

.upload-icon {
  font-size: 52px;
}

.upload-hint {
  margin-top: 12px;
  font-size: 12px !important;
  color: #4b5563 !important;
}

/* Workspace */
.workspace {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.image-container {
  width: 100%;
  max-width: 380px;
}

.image-label {
  text-align: center;
  margin-bottom: 6px;
  font-size: 14px;
  font-weight: 500;
  color: #a5b4fc;
}

.image-wrapper {
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  background: #111827;
  border: 1px solid rgba(99, 102, 241, 0.2);
  box-shadow: 0 0 30px rgba(99, 102, 241, 0.1);
}

.image-wrapper img {
  width: 100%;
  display: block;
}

.img-base {
  position: relative;
  z-index: 1;
}

.img-result {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 2;
  clip-path: inset(100% 0 0 0);
}

.img-result.img-reveal {
  animation: revealBottomToTop 1.5s ease-out forwards;
}

.img-result.img-done {
  clip-path: inset(0 0 0 0);
}

@keyframes revealBottomToTop {
  0% {
    clip-path: inset(100% 0 0 0);
  }
  100% {
    clip-path: inset(0 0 0 0);
  }
}

.image-wrapper.processing {
  animation: pulse-border 1.5s ease-in-out infinite;
}

@keyframes pulse-border {
  0%, 100% { box-shadow: 0 0 20px rgba(99, 102, 241, 0.2); }
  50% { box-shadow: 0 0 40px rgba(99, 102, 241, 0.5); }
}

/* Scan Line */
.scan-line {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, #818cf8, #c084fc, #818cf8, transparent);
  animation: scan 2s linear infinite;
  box-shadow: 0 0 20px 4px rgba(129, 140, 248, 0.4);
}

@keyframes scan {
  0% { top: 0; }
  100% { top: 100%; }
}

/* Processing Overlay */
.processing-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
}

.spinner {
  width: 48px;
  height: 48px;
  border: 4px solid rgba(99, 102, 241, 0.2);
  border-top-color: #818cf8;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Progress */
.progress-section {
  width: 100%;
  max-width: 380px;
}

.progress-bar-container {
  width: 100%;
  height: 8px;
  background: #1f2937;
  border-radius: 4px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #818cf8, #c084fc);
  border-radius: 4px;
  transition: width 0.1s linear;
}

.progress-text {
  text-align: center;
  margin-top: 12px;
  font-size: 15px;
  font-weight: 500;
  color: #e0e6f0;
}

.tech-log {
  text-align: center;
  margin-top: 8px;
  font-size: 12px;
  color: #4b5563;
  font-family: 'Courier New', monospace;
}

/* Buttons */
.actions {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 28px;
}

.btn-primary {
  padding: 14px 36px;
  font-size: 18px;
  font-weight: 700;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  color: white;
  transition: all 0.3s ease;
  font-family: inherit;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(99, 102, 241, 0.4);
}

.btn-primary:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.btn-glow {
  animation: glow 2s ease-in-out infinite;
}

@keyframes glow {
  0%, 100% { box-shadow: 0 0 15px rgba(99, 102, 241, 0.3); }
  50% { box-shadow: 0 0 30px rgba(99, 102, 241, 0.6); }
}

.btn-success {
  padding: 8px 24px;
  font-size: 14px;
  font-weight: 700;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  background: linear-gradient(135deg, #059669, #10b981);
  color: white;
  transition: all 0.3s ease;
  font-family: inherit;
}

.btn-success:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(16, 185, 129, 0.4);
}

/* Result Info */
.result-info {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 12px 16px;
  background: rgba(16, 185, 129, 0.1);
  border: 1px solid rgba(16, 185, 129, 0.3);
  border-radius: 12px;
  max-width: 380px;
  width: 100%;
}

.success-icon {
  font-size: 24px;
}

.success-text {
  font-size: 22px;
  font-weight: 700;
  color: #6ee7b7;
}

/* Footer */
.footer {
  padding: 8px;
  text-align: center;
  border-top: 1px solid rgba(99, 102, 241, 0.1);
}

.footer p {
  font-size: 11px;
  color: #374151;
  letter-spacing: 0.5px;
}

/* Responsive */
@media (max-width: 640px) {
  .header-content {
    flex-direction: column;
    align-items: flex-start;
  }
  .upload-zone {
    padding: 30px 20px;
  }
  .btn-primary, .btn-success {
    width: 100%;
    padding: 14px 20px;
  }
}
</style>
