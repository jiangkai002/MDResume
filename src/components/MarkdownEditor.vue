<script setup>
import { ref, computed, nextTick } from 'vue'

// ── Markdown 语法高亮 ──────────────────────────────────────────────
const escHtml = (s) =>
  s.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')

const sp = (cls, text) => `<span class="${cls}">${text}</span>`

const applyInline = (line) => {
  // 粗斜体 ***
  line = line.replace(/(\*{3})((?:[^*]|\*(?!\*\*))+?)(\*{3})/g,
    (_, a, m, b) => sp('hl-mark', a) + sp('hl-bold hl-italic', m) + sp('hl-mark', b))
  // 粗体 ** / __
  line = line.replace(/(\*{2}|_{2})((?:[^*_\n])+?)(\*{2}|_{2})/g,
    (_, a, m, b) => sp('hl-mark', a) + sp('hl-bold', m) + sp('hl-mark', b))
  // 斜体 * / _
  line = line.replace(/(?<!\*)\*([^*\n]+?)\*(?!\*)/g,
    (_, m) => sp('hl-mark', '*') + sp('hl-italic', m) + sp('hl-mark', '*'))
  // 删除线 ~~
  line = line.replace(/(~~)((?:[^~]|~(?!~))+?)(~~)/g,
    (_, a, m, b) => sp('hl-mark', a) + sp('hl-strike', m) + sp('hl-mark', b))
  // 图片 / 链接 ![](url) [](url)
  line = line.replace(/(!\[|\[)([^\]]*?)(\]\()([^)]*?)(\))/g,
    (_, a, lt, c, url, e) =>
      sp('hl-mark', a) + sp('hl-link-text', lt) + sp('hl-mark', c) + sp('hl-link-url', url) + sp('hl-mark', e))
  // 行内代码 `code`
  line = line.replace(/(`+)([^`]*?)(`+)/g,
    (_, a, m, b) => sp('hl-code-mark', a) + sp('hl-code', m) + sp('hl-code-mark', b))
  return line
}

const highlightMarkdown = (text) => {
  if (!text) return ''
  const lines = escHtml(text).split('\n')
  let inFence = false
  return lines.map((line) => {
    // 代码围栏开合
    const fenceMatch = line.match(/^(```)([\w-]*)(.*)$/)
    if (fenceMatch) {
      inFence = !inFence
      return sp('hl-fence-mark', fenceMatch[1] + fenceMatch[2]) + fenceMatch[3]
    }
    if (inFence) return sp('hl-fence-body', line)

    // 标题
    const hm = line.match(/^(#{1,6})([ \t].*)$/)
    if (hm) return sp('hl-heading-mark', hm[1]) + sp('hl-heading', applyInline(hm[2]))

    // 分隔线 --- / *** / ___
    if (/^[ \t]*([-*_][ \t]*){3,}$/.test(line)) return sp('hl-hr', line)

    // 引用 >（转义后 &gt;）
    if (/^[ \t]*&gt;/.test(line)) {
      return line.replace(/^([ \t]*&gt;)(.*)/, (_, q, rest) =>
        sp('hl-quote-mark', q) + sp('hl-quote', applyInline(rest)))
    }

    // 有序列表
    const ol = line.match(/^([ \t]*)(\d+\.)( .*)$/)
    if (ol) return ol[1] + sp('hl-list-mark', ol[2]) + applyInline(ol[3])

    // 无序列表
    const ul = line.match(/^([ \t]*)([-*+])( .*)$/)
    if (ul) return ul[1] + sp('hl-list-mark', ul[2]) + applyInline(ul[3])

    return applyInline(line)
  }).join('\n')
}
// ─────────────────────────────────────────────────────────────────

const props = defineProps({
  modelValue: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['update:modelValue'])

const textareaRef = ref(null)
const highlightLayerRef = ref(null)

const content = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val),
})

const lineCount = computed(() => {
  return (content.value || '').split('\n').length
})

const highlightedContent = computed(() => highlightMarkdown(content.value))

const wrapSelection = (before, after = '') => {
  const el = textareaRef.value
  if (!el) return
  const start = el.selectionStart
  const end = el.selectionEnd
  const selected = content.value.substring(start, end)
  const newContent =
    content.value.substring(0, start) +
    before +
    selected +
    after +
    content.value.substring(end)
  content.value = newContent
  nextTick(() => {
    el.focus()
    if (selected) {
      el.setSelectionRange(start + before.length, start + before.length + selected.length)
    } else {
      el.setSelectionRange(start + before.length, start + before.length)
    }
  })
}

const insertLinePrefix = (prefix) => {
  const el = textareaRef.value
  if (!el) return
  const start = el.selectionStart
  const end = el.selectionEnd
  const lines = content.value.split('\n')
  let charCount = 0
  let startLine = 0
  let endLine = 0
  for (let i = 0; i < lines.length; i++) {
    if (charCount + lines[i].length >= start && startLine === 0) startLine = i
    if (charCount + lines[i].length >= end) { endLine = i; break }
    charCount += lines[i].length + 1
  }
  for (let i = startLine; i <= endLine; i++) {
    if (lines[i].startsWith(prefix)) {
      lines[i] = lines[i].substring(prefix.length)
    } else {
      lines[i] = prefix + lines[i]
    }
  }
  content.value = lines.join('\n')
  nextTick(() => el.focus())
}

const insertBlock = (text, cursorOffset = 0) => {
  const el = textareaRef.value
  if (!el) return
  const start = el.selectionStart
  const newContent = content.value.substring(0, start) + text + content.value.substring(start)
  content.value = newContent
  nextTick(() => {
    el.focus()
    el.setSelectionRange(start + cursorOffset, start + cursorOffset)
  })
}

const handleTab = (e) => {
  e.preventDefault()
  const el = textareaRef.value
  const start = el.selectionStart
  const end = el.selectionEnd
  const newContent = content.value.substring(0, start) + '  ' + content.value.substring(end)
  content.value = newContent
  nextTick(() => {
    el.focus()
    el.setSelectionRange(start + 2, start + 2)
  })
}

const toolbarGroups = [
  {
    items: [
      { label: 'H1', title: '一级标题', action: () => insertLinePrefix('# ') },
      { label: 'H2', title: '二级标题', action: () => insertLinePrefix('## ') },
      { label: 'H3', title: '三级标题', action: () => insertLinePrefix('### ') },
    ],
  },
  {
    items: [
      {
        icon: 'bold',
        title: '粗体 (Ctrl+B)',
        action: () => wrapSelection('**', '**'),
      },
      {
        icon: 'italic',
        title: '斜体 (Ctrl+I)',
        action: () => wrapSelection('*', '*'),
      },
      {
        icon: 'strikethrough',
        title: '删除线',
        action: () => wrapSelection('~~', '~~'),
      },
    ],
  },
  {
    items: [
      {
        icon: 'list-ul',
        title: '无序列表',
        action: () => insertLinePrefix('- '),
      },
      {
        icon: 'list-ol',
        title: '有序列表',
        action: () => insertLinePrefix('1. '),
      },
      {
        icon: 'quote',
        title: '引用',
        action: () => insertLinePrefix('> '),
      },
    ],
  },
  {
    items: [
      {
        icon: 'code',
        title: '行内代码',
        action: () => wrapSelection('`', '`'),
      },
      {
        icon: 'code-block',
        title: '代码块',
        action: () => insertBlock('\n```\n\n```\n', 5),
      },
      {
        icon: 'hr',
        title: '分隔线',
        action: () => insertBlock('\n\n---\n\n', 6),
      },
    ],
  },
  {
    items: [
      {
        icon: 'link',
        title: '链接',
        action: () => wrapSelection('[', '](url)'),
      },
      {
        icon: 'table',
        title: '插入表格',
        action: () =>
          insertBlock(
            '\n| 列1 | 列2 | 列3 |\n|-----|-----|-----|\n| 内容 | 内容 | 内容 |\n',
            1,
          ),
      },
    ],
  },
]

const handleKeydown = (e) => {
  if (e.key === 'Tab') {
    handleTab(e)
    return
  }
  if (e.ctrlKey || e.metaKey) {
    if (e.key === 'b') {
      e.preventDefault()
      wrapSelection('**', '**')
    } else if (e.key === 'i') {
      e.preventDefault()
      wrapSelection('*', '*')
    }
  }
}

const syncScroll = () => {
  const ta = textareaRef.value
  const hl = highlightLayerRef.value
  if (!ta || !hl) return
  hl.scrollTop = ta.scrollTop
  hl.scrollLeft = ta.scrollLeft
}
</script>

<template>
  <div class="editor-wrapper">
    <div class="editor-toolbar">
      <template v-for="(group, gi) in toolbarGroups" :key="gi">
        <div class="toolbar-divider" v-if="gi > 0"></div>
        <button
          v-for="item in group.items"
          :key="item.title"
          class="toolbar-btn"
          :title="item.title"
          @click="item.action"
          @mousedown.prevent
        >
          <!-- H1/H2/H3 文字标签 -->
          <span v-if="item.label" class="toolbar-label">{{ item.label }}</span>

          <!-- Bold -->
          <svg v-else-if="item.icon === 'bold'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <path d="M6 4h8a4 4 0 0 1 4 4 4 4 0 0 1-4 4H6z"/>
            <path d="M6 12h9a4 4 0 0 1 4 4 4 4 0 0 1-4 4H6z"/>
          </svg>

          <!-- Italic -->
          <svg v-else-if="item.icon === 'italic'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <line x1="19" y1="4" x2="10" y2="4"/>
            <line x1="14" y1="20" x2="5" y2="20"/>
            <line x1="15" y1="4" x2="9" y2="20"/>
          </svg>

          <!-- Strikethrough -->
          <svg v-else-if="item.icon === 'strikethrough'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="5" y1="12" x2="19" y2="12"/>
            <path d="M16 6C16 6 14.5 4 12 4C9.5 4 7 5.5 7 8C7 10.5 10 11.5 12 12"/>
            <path d="M8 18C8 18 9.5 20 12 20C14.5 20 17 18.5 17 16C17 13.5 14 12.5 12 12"/>
          </svg>

          <!-- Unordered list -->
          <svg v-else-if="item.icon === 'list-ul'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="9" y1="6" x2="20" y2="6"/>
            <line x1="9" y1="12" x2="20" y2="12"/>
            <line x1="9" y1="18" x2="20" y2="18"/>
            <circle cx="4" cy="6" r="1.5" fill="currentColor" stroke="none"/>
            <circle cx="4" cy="12" r="1.5" fill="currentColor" stroke="none"/>
            <circle cx="4" cy="18" r="1.5" fill="currentColor" stroke="none"/>
          </svg>

          <!-- Ordered list -->
          <svg v-else-if="item.icon === 'list-ol'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="10" y1="6" x2="21" y2="6"/>
            <line x1="10" y1="12" x2="21" y2="12"/>
            <line x1="10" y1="18" x2="21" y2="18"/>
            <text x="2" y="8" font-size="7" fill="currentColor" stroke="none" font-family="monospace">1.</text>
            <text x="2" y="14" font-size="7" fill="currentColor" stroke="none" font-family="monospace">2.</text>
            <text x="2" y="20" font-size="7" fill="currentColor" stroke="none" font-family="monospace">3.</text>
          </svg>

          <!-- Quote -->
          <svg v-else-if="item.icon === 'quote'" viewBox="0 0 24 24" fill="currentColor">
            <path d="M3 21c3 0 7-1 7-8V5c0-1.25-.756-2.017-2-2H4c-1.25 0-2 .75-2 1.972V11c0 1.25.75 2 2 2 1 0 1 0 1 1v1c0 1-1 2-2 2s-1 .008-1 1.031V20c0 1 0 1 1 1z"/>
            <path d="M15 21c3 0 7-1 7-8V5c0-1.25-.757-2.017-2-2h-4c-1.25 0-2 .75-2 1.972V11c0 1.25.75 2 2 2h.75c0 2.25.25 4-2.75 4v3c0 1 0 1 1 1z"/>
          </svg>

          <!-- Inline code -->
          <svg v-else-if="item.icon === 'code'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="16 18 22 12 16 6"/>
            <polyline points="8 6 2 12 8 18"/>
          </svg>

          <!-- Code block -->
          <svg v-else-if="item.icon === 'code-block'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="2" y="3" width="20" height="18" rx="2"/>
            <polyline points="8 9 12 13 8 17" stroke-linecap="round" stroke-linejoin="round"/>
            <line x1="14" y1="17" x2="18" y2="17" stroke-linecap="round"/>
          </svg>

          <!-- HR -->
          <svg v-else-if="item.icon === 'hr'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="3" y1="12" x2="21" y2="12"/>
            <line x1="3" y1="6" x2="10" y2="6" stroke-width="1.5" opacity="0.5"/>
            <line x1="14" y1="6" x2="21" y2="6" stroke-width="1.5" opacity="0.5"/>
            <line x1="3" y1="18" x2="10" y2="18" stroke-width="1.5" opacity="0.5"/>
            <line x1="14" y1="18" x2="21" y2="18" stroke-width="1.5" opacity="0.5"/>
          </svg>

          <!-- Link -->
          <svg v-else-if="item.icon === 'link'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"/>
            <path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"/>
          </svg>

          <!-- Table -->
          <svg v-else-if="item.icon === 'table'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="3" width="18" height="18" rx="2"/>
            <line x1="3" y1="9" x2="21" y2="9"/>
            <line x1="3" y1="15" x2="21" y2="15"/>
            <line x1="9" y1="3" x2="9" y2="21"/>
            <line x1="15" y1="3" x2="15" y2="21"/>
          </svg>
        </button>
      </template>
    </div>

    <div class="editor-body">
      <div class="line-numbers" aria-hidden="true">
        <div v-for="n in lineCount" :key="n" class="line-number">{{ n }}</div>
      </div>
      <div class="editor-content">
        <div
          ref="highlightLayerRef"
          class="highlight-layer"
          v-html="highlightedContent"
          aria-hidden="true"
        ></div>
        <textarea
          ref="textareaRef"
          class="editor-textarea"
          v-model="content"
          @keydown="handleKeydown"
          @scroll="syncScroll"
          spellcheck="false"
          placeholder="在此输入 Markdown 内容..."
        ></textarea>
      </div>
    </div>

    <div class="editor-statusbar">
      <span>行数：{{ lineCount }}</span>
      <span>字符：{{ content.length }}</span>
    </div>
  </div>
</template>

<style scoped>
.editor-wrapper {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: var(--bg-editor);
  border-right: 1px solid var(--border);
}

.editor-toolbar {
  display: flex;
  align-items: center;
  padding: 6px 10px;
  background: var(--bg-toolbar);
  border-bottom: 1px solid var(--border);
  gap: 2px;
  flex-wrap: wrap;
  flex-shrink: 0;
}

.toolbar-divider {
  width: 1px;
  height: 20px;
  background: var(--border);
  margin: 0 4px;
}

.toolbar-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 28px;
  border: none;
  background: transparent;
  color: var(--toolbar-icon);
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.15s;
  padding: 0;
}

.toolbar-btn svg {
  width: 15px;
  height: 15px;
}

.toolbar-label {
  font-size: 11px;
  font-weight: 700;
  font-family: monospace;
}

.toolbar-btn:hover {
  background: var(--toolbar-hover-bg);
  color: var(--toolbar-hover-color);
}

.toolbar-btn:active {
  background: var(--toolbar-active-bg);
  color: var(--text-primary);
}

.editor-body {
  display: flex;
  flex: 1;
  overflow: hidden;
  min-height: 0;
}

.line-numbers {
  padding: 14px 0;
  width: 46px;
  background: var(--bg-linenums);
  border-right: 1px solid var(--border-subtle);
  overflow: hidden;
  flex-shrink: 0;
  user-select: none;
}

.line-number {
  height: 21px;
  line-height: 21px;
  font-size: 12px;
  font-family: 'Consolas', 'Fira Code', 'Courier New', monospace;
  color: var(--text-faint);
  text-align: right;
  padding-right: 10px;
}

/* 编辑区容器（相对定位，用于叠加层） */
.editor-content {
  flex: 1;
  position: relative;
  overflow: hidden;
  min-width: 0;
}

/* 语法高亮叠加层 */
.highlight-layer {
  position: absolute;
  inset: 0;
  padding: 14px 16px;
  font-family: 'Consolas', 'Fira Code', 'Courier New', monospace;
  font-size: 13.5px;
  line-height: 21px;
  white-space: pre;
  overflow: hidden;
  pointer-events: none;
  color: var(--text-editor);
  word-break: normal;
  tab-size: 2;
  margin: 0;
  border: none;
}

/* textarea 透明覆盖在高亮层上方 */
.editor-textarea {
  position: absolute;
  inset: 0;
  padding: 14px 16px;
  background: transparent;
  color: transparent;
  caret-color: var(--text-editor);
  border: none;
  outline: none;
  resize: none;
  font-family: 'Consolas', 'Fira Code', 'Courier New', monospace;
  font-size: 13.5px;
  line-height: 21px;
  overflow-y: auto;
  overflow-x: auto;
  white-space: pre;
  tab-size: 2;
  z-index: 1;
}

.editor-textarea::placeholder {
  color: var(--text-faint);
}

/* 选区高亮（半透明使高亮层可见） */
.editor-textarea::selection {
  background: rgba(100, 150, 220, 0.35);
  color: transparent;
}

.editor-statusbar {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 4px 16px;
  background: var(--bg-statusbar);
  border-top: 1px solid var(--border-subtle);
  font-size: 11px;
  color: var(--text-muted);
  flex-shrink: 0;
}
</style>

<style>
/* 高亮层语法颜色（非 scoped，作用于 v-html 渲染内容） */
.hl-heading-mark { color: var(--hl-heading-mark); font-weight: 700; }
.hl-heading      { color: var(--hl-heading); font-weight: 600; }
.hl-mark         { color: var(--hl-mark); }
.hl-bold         { color: var(--hl-bold); font-weight: 700; }
.hl-italic       { color: var(--hl-italic); font-style: italic; }
.hl-strike       { color: var(--hl-strike); text-decoration: line-through; }
.hl-code         { color: var(--hl-code); }
.hl-code-mark    { color: var(--hl-code-mark); }
.hl-link-text    { color: var(--hl-link-text); }
.hl-link-url     { color: var(--hl-link-url); }
.hl-quote-mark   { color: var(--hl-quote-mark); font-weight: 600; }
.hl-quote        { color: var(--hl-quote); }
.hl-list-mark    { color: var(--hl-list-mark); font-weight: 600; }
.hl-hr           { color: var(--hl-hr); }
.hl-fence-mark   { color: var(--hl-fence-mark); font-weight: 600; }
.hl-fence-body   { color: var(--hl-fence-body); }
</style>
