<script setup>
import { ref, computed, nextTick } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['update:modelValue'])

const textareaRef = ref(null)

const content = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val),
})

const lineCount = computed(() => {
  return (content.value || '').split('\n').length
})

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

const syncScroll = (e) => {
  // 可选：同步滚动
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

.editor-textarea {
  flex: 1;
  padding: 14px 16px;
  background: transparent;
  color: var(--text-editor);
  border: none;
  outline: none;
  resize: none;
  font-family: 'Consolas', 'Fira Code', 'Courier New', monospace;
  font-size: 13.5px;
  line-height: 21px;
  overflow-y: auto;
  white-space: pre;
  overflow-x: auto;
  tab-size: 2;
}

.editor-textarea::placeholder {
  color: var(--text-faint);
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
