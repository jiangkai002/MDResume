<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const isDark = ref(true)
const toggleTheme = () => {
  isDark.value = !isDark.value
  document.documentElement.setAttribute('data-theme', isDark.value ? 'dark' : 'light')
  localStorage.setItem('cv-editor-theme', isDark.value ? 'dark' : 'light')
}
const initTheme = () => {
  const saved = localStorage.getItem('cv-editor-theme')
  isDark.value = saved ? saved === 'dark' : true
  document.documentElement.setAttribute('data-theme', isDark.value ? 'dark' : 'light')
}
import MarkdownEditor from './components/MarkdownEditor.vue'
import MarkdownPreview from './components/MarkdownPreview.vue'
import defaultContent from '../CV.md?raw'

const content = ref(defaultContent)
const previewRef = ref(null)
const layoutRef = ref(null)
const editorWidth = ref(50)
const isResizing = ref(false)
const isExporting = ref(false)

const startResize = (e) => {
  isResizing.value = true
  document.body.style.cursor = 'col-resize'
  document.body.style.userSelect = 'none'
  e.preventDefault()
}

const onResize = (e) => {
  if (!isResizing.value || !layoutRef.value) return
  const rect = layoutRef.value.getBoundingClientRect()
  const pct = ((e.clientX - rect.left) / rect.width) * 100
  editorWidth.value = Math.min(Math.max(pct, 20), 80)
}

const stopResize = () => {
  if (!isResizing.value) return
  isResizing.value = false
  document.body.style.cursor = ''
  document.body.style.userSelect = ''
}

onMounted(() => {
  initTheme()
  document.addEventListener('mousemove', onResize)
  document.addEventListener('mouseup', stopResize)
})

onUnmounted(() => {
  document.removeEventListener('mousemove', onResize)
  document.removeEventListener('mouseup', stopResize)
})

const handleImport = () => {
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = '.md,.markdown,.txt'
  input.onchange = (e) => {
    const file = e.target.files[0]
    if (!file) return
    const reader = new FileReader()
    reader.onload = (evt) => {
      content.value = evt.target.result
    }
    reader.readAsText(file, 'UTF-8')
  }
  input.click()
}

const handleExportMD = () => {
  const blob = new Blob([content.value], { type: 'text/markdown;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'resume.md'
  a.click()
  URL.revokeObjectURL(url)
}

const handleExportPDF = async () => {
  if (isExporting.value) return
  isExporting.value = true

  try {
    const { default: html2pdf } = await import('html2pdf.js')
    const element = previewRef.value?.getPreviewElement()
    if (!element) return

    const opt = {
      margin: [8, 8, 8, 8],
      filename: 'resume.pdf',
      image: { type: 'jpeg', quality: 0.98 },
      html2canvas: {
        scale: 2,
        useCORS: true,
        letterRendering: true,
        logging: false,
      },
      jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' },
    }

    await html2pdf().set(opt).from(element).save()
  } finally {
    isExporting.value = false
  }
}
</script>

<template>
  <div class="app">
    <header class="app-header">
      <div class="header-left">
        <svg class="logo-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" stroke="#4a86c8" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          <polyline points="14 2 14 8 20 8" stroke="#4a86c8" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          <line x1="16" y1="13" x2="8" y2="13" stroke="#4a86c8" stroke-width="2" stroke-linecap="round"/>
          <line x1="16" y1="17" x2="8" y2="17" stroke="#4a86c8" stroke-width="2" stroke-linecap="round"/>
          <polyline points="10 9 9 9 8 9" stroke="#4a86c8" stroke-width="2" stroke-linecap="round"/>
        </svg>
        <span class="app-title">简历 Markdown 编辑器</span>
      </div>
      <div class="header-actions">
        <button class="btn btn-icon" @click="toggleTheme" :title="isDark ? '切换到亮色模式' : '切换到暗色模式'">
          <!-- 暗色时显示太阳（切到亮色），亮色时显示月亮（切到暗色） -->
          <svg v-if="isDark" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="5"/>
            <line x1="12" y1="1" x2="12" y2="3"/>
            <line x1="12" y1="21" x2="12" y2="23"/>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
            <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
            <line x1="1" y1="12" x2="3" y2="12"/>
            <line x1="21" y1="12" x2="23" y2="12"/>
            <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
            <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
          </svg>
        </button>
        <div class="header-divider"></div>
        <button class="btn btn-ghost" @click="handleImport" title="导入 Markdown 文件">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
            <polyline points="17 8 12 3 7 8"/>
            <line x1="12" y1="3" x2="12" y2="15"/>
          </svg>
          导入 MD
        </button>
        <button class="btn btn-ghost" @click="handleExportMD" title="导出 Markdown 文件">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
            <polyline points="7 10 12 15 17 10"/>
            <line x1="12" y1="15" x2="12" y2="3"/>
          </svg>
          导出 MD
        </button>
        <button
          class="btn btn-primary"
          :class="{ loading: isExporting }"
          @click="handleExportPDF"
          :disabled="isExporting"
          title="导出 PDF 文件"
        >
          <svg v-if="!isExporting" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="6 9 6 2 18 2 18 9"/>
            <path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/>
            <rect x="6" y="14" width="12" height="8"/>
          </svg>
          <span v-if="isExporting" class="spinner"></span>
          {{ isExporting ? '生成中...' : '导出 PDF' }}
        </button>
      </div>
    </header>

    <div class="editor-layout" ref="layoutRef">
      <div class="pane editor-pane" :style="{ width: editorWidth + '%' }">
        <MarkdownEditor v-model="content" />
      </div>

      <div
        class="resize-handle"
        :class="{ active: isResizing }"
        @mousedown="startResize"
      >
        <div class="resize-grip"></div>
      </div>

      <div class="pane preview-pane" :style="{ flex: 1 }">
        <MarkdownPreview :content="content" ref="previewRef" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: var(--bg-base);
}

.app-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  height: 52px;
  background: var(--bg-header);
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
  box-shadow: 0 1px 0 var(--border);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo-icon {
  width: 24px;
  height: 24px;
}

.app-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: 0.3px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.header-divider {
  width: 1px;
  height: 20px;
  background: var(--border);
  margin: 0 2px;
}

.btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 14px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  border: 1px solid transparent;
  transition: all 0.2s;
  outline: none;
}

.btn svg {
  width: 14px;
  height: 14px;
  flex-shrink: 0;
}

.btn-icon {
  width: 32px;
  height: 32px;
  padding: 0;
  border-radius: 6px;
  background: transparent;
  border: 1px solid var(--btn-ghost-border);
  color: var(--btn-ghost-color);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  outline: none;
}

.btn-icon svg {
  width: 15px;
  height: 15px;
}

.btn-icon:hover {
  background: var(--btn-ghost-hover-bg);
  border-color: var(--btn-ghost-hover-border);
  color: var(--btn-ghost-hover-color);
}

.btn-ghost {
  background: transparent;
  border-color: var(--btn-ghost-border);
  color: var(--btn-ghost-color);
}

.btn-ghost:hover {
  background: var(--btn-ghost-hover-bg);
  border-color: var(--btn-ghost-hover-border);
  color: var(--btn-ghost-hover-color);
}

.btn-primary {
  background: var(--accent);
  border-color: var(--accent);
  color: #fff;
}

.btn-primary:hover:not(:disabled) {
  background: var(--accent-hover);
  border-color: var(--accent-hover);
}

.btn-primary:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.btn-primary.loading {
  background: var(--accent-dim);
  border-color: var(--accent-dim);
}

.spinner {
  width: 12px;
  height: 12px;
  border: 2px solid rgba(255,255,255,0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  flex-shrink: 0;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.editor-layout {
  display: flex;
  flex: 1;
  overflow: hidden;
  min-height: 0;
}

.pane {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-width: 0;
}

.editor-pane {
  flex-shrink: 0;
}

.resize-handle {
  width: 5px;
  background: var(--resize-bg);
  cursor: col-resize;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
  flex-shrink: 0;
  position: relative;
}

.resize-handle:hover,
.resize-handle.active {
  background: var(--accent);
}

.resize-grip {
  width: 2px;
  height: 40px;
  background: var(--resize-grip);
  border-radius: 2px;
}

.resize-handle:hover .resize-grip,
.resize-handle.active .resize-grip {
  background: var(--resize-grip-active);
}
</style>
