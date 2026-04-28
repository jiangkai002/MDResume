<script setup>
import { ref, computed } from 'vue'
import { marked, Renderer } from 'marked'
import hljs from 'highlight.js'

const props = defineProps({
  content: {
    type: String,
    default: '',
  },
})

const previewContentRef = ref(null)

const renderer = new Renderer()

renderer.code = ({ text, lang }) => {
  const language = lang && hljs.getLanguage(lang) ? lang : 'plaintext'
  const highlighted = hljs.highlight(text, { language }).value
  return `<pre><code class="hljs language-${language}">${highlighted}</code></pre>`
}

marked.use({
  renderer,
  gfm: true,
  breaks: true,
})

const renderedContent = computed(() => {
  try {
    return marked.parse(props.content || '')
  } catch {
    return '<p style="color:red">Markdown 解析错误</p>'
  }
})

const getPreviewElement = () => previewContentRef.value

defineExpose({ getPreviewElement })
</script>

<template>
  <div class="preview-wrapper">
    <div class="preview-header">
      <span class="preview-label">预览</span>
      <span class="preview-hint">A4 预览效果</span>
    </div>
    <div class="preview-scroll">
      <div class="preview-paper">
        <div
          ref="previewContentRef"
          class="preview-content"
          v-html="renderedContent"
        ></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.preview-wrapper {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: var(--bg-preview);
}

.preview-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 6px 16px;
  background: var(--preview-header-bg);
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
  height: 41px;
}

.preview-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--preview-label-color);
  text-transform: uppercase;
  letter-spacing: 0.8px;
}

.preview-hint {
  font-size: 11px;
  color: var(--text-faint);
}

.preview-scroll {
  flex: 1;
  overflow-y: auto;
  padding: 24px 16px;
  background: var(--bg-preview);
}

.preview-paper {
  background: #ffffff;
  max-width: 794px;
  min-height: 1123px;
  margin: 0 auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
  border-radius: 2px;
}

.preview-content {
  padding: 40px 48px;
  color: #333;
  font-family: 'Microsoft YaHei', 'Arial', sans-serif;
  font-size: 14px;
  line-height: 1.7;
}
</style>

<style>
/* 全局预览内容样式（非 scoped，用于渲染 v-html 内容） */
.preview-content h1 {
  font-size: 24px;
  font-weight: 700;
  color: #1a1a2e;
  margin: 0 0 12px 0;
  padding-bottom: 8px;
  border-bottom: 2px solid #4a86c8;
}

.preview-content h2 {
  font-size: 16px;
  font-weight: 700;
  color: #4a86c8;
  margin: 20px 0 10px 0;
  padding-bottom: 4px;
  border-bottom: 1px solid #e0e8f5;
}

.preview-content h3 {
  font-size: 14px;
  font-weight: 600;
  color: #2a4a7a;
  margin: 14px 0 6px 0;
}

.preview-content p {
  margin: 6px 0;
  color: #333;
}

.preview-content ul,
.preview-content ol {
  margin: 6px 0;
  padding-left: 24px;
}

.preview-content li {
  margin: 4px 0;
  color: #444;
}

.preview-content strong {
  font-weight: 600;
  color: #1a1a2e;
}

.preview-content em {
  font-style: italic;
  color: #555;
}

.preview-content a {
  color: #4a86c8;
  text-decoration: none;
}

.preview-content a:hover {
  text-decoration: underline;
}

.preview-content blockquote {
  margin: 10px 0;
  padding: 8px 16px;
  border-left: 3px solid #4a86c8;
  background: #f0f5ff;
  color: #555;
  border-radius: 0 4px 4px 0;
}

.preview-content code {
  font-family: 'Consolas', 'Fira Code', 'Courier New', monospace;
  font-size: 12px;
  background: #f0f0f5;
  color: #c7254e;
  padding: 1px 5px;
  border-radius: 3px;
}

.preview-content pre {
  margin: 10px 0;
  padding: 14px 16px;
  background: #1e1e2e;
  border-radius: 6px;
  overflow-x: auto;
}

.preview-content pre code {
  background: transparent;
  color: #abb2bf;
  padding: 0;
  font-size: 12px;
}

.preview-content table {
  width: 100%;
  border-collapse: collapse;
  margin: 10px 0;
  font-size: 13px;
}

.preview-content th {
  background: #f0f4fa;
  color: #2a4a7a;
  font-weight: 600;
  text-align: left;
  padding: 8px 12px;
  border: 1px solid #d5e0f0;
}

.preview-content td {
  padding: 6px 12px;
  border: 1px solid #e0e8f5;
  color: #444;
  vertical-align: top;
}

.preview-content tr:nth-child(even) td {
  background: #f8fafd;
}

.preview-content hr {
  border: none;
  border-top: 1.5px solid #dde6f0;
  margin: 16px 0;
}

.preview-content img {
  max-width: 100%;
  border-radius: 4px;
}

/* highlight.js 基础主题 */
.hljs {
  color: #abb2bf;
}
.hljs-comment, .hljs-quote { color: #5c6370; font-style: italic; }
.hljs-keyword, .hljs-selector-tag { color: #c678dd; }
.hljs-string, .hljs-attr { color: #98c379; }
.hljs-number, .hljs-literal { color: #d19a66; }
.hljs-built_in, .hljs-type { color: #e5c07b; }
.hljs-function, .hljs-title { color: #61afef; }
.hljs-variable, .hljs-params { color: #e06c75; }
.hljs-tag { color: #e06c75; }
.hljs-name { color: #e06c75; }
</style>
