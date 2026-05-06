# MDResume

一个基于 Vue 3 的 Markdown 简历编辑器。

项目目标很直接：把「写简历」和「排版预览」放到同一个界面里，左侧专注编辑 Markdown，右侧实时查看 A4 纸张效果，同时支持导入、导出和 PDF 生成，适合做个人简历、简短介绍页或需要打印输出的 Markdown 文档。

## 项目特点

- 双栏布局，左侧编辑、右侧预览，适合边写边调。
- 支持拖拽分栏宽度，按自己的习惯调整编辑区和预览区比例。
- 内置 Markdown 工具栏，常用标题、列表、引用、代码块、表格、链接等操作可以直接插入。
- 编辑区提供行号和基础语法高亮，输入时更接近代码编辑器的体验。
- 预览区使用 `marked` 解析 Markdown，并结合 `highlight.js` 渲染代码高亮。
- 预览内容按 A4 页面风格排版，更方便直接导出为 PDF。
- 支持深色 / 浅色主题切换，并会记住上一次使用的主题。
- 支持导入本地 Markdown 文件，也可以导出为 `.md` 或 `.pdf`。

## 适用场景

- 用 Markdown 编写个人简历
- 快速生成可打印的自我介绍文档
- 需要一边写一边看排版结果的轻量编辑场景

## 技术栈

- Vue 3
- Vite
- marked
- highlight.js
- html2pdf.js

## 快速开始

先安装依赖：

```bash
npm install
```

启动开发环境：

```bash
npm run dev
```

构建生产版本：

```bash
npm run build
```

本地预览构建结果：

```bash
npm run preview
```

## 使用说明

1. 在左侧编辑器输入或粘贴 Markdown 内容。
2. 通过顶部工具栏快速插入标题、粗体、列表、引用、代码块、表格等常见语法。
3. 在右侧查看实时预览效果，预览区域按 A4 页面样式展示。
4. 如需处理已有文件，可使用顶部按钮导入本地 Markdown。
5. 编辑完成后，可以导出为 Markdown 文件，或直接导出为 PDF。

## 目录结构

```text
MDResume
├─ public
├─ src
│  ├─ components
│  │  ├─ MarkdownEditor.vue
│  │  └─ MarkdownPreview.vue
│  ├─ App.vue
│  ├─ main.js
│  └─ style.css
├─ index.html
├─ package.json
├─ tsconfig.json
└─ vite.config.js
```

## 核心设计

### `MarkdownEditor.vue`

负责编辑区能力，包括：

- 工具栏操作
- 行号显示
- 文本输入
- 基础 Markdown 语法高亮
- 快捷键处理

### `MarkdownPreview.vue`

负责预览区能力，包括：

- Markdown 解析
- 代码高亮
- A4 页面样式渲染
- 暴露预览 DOM，供 PDF 导出使用

### `App.vue`

负责整体页面编排和应用级功能，包括：

- 深浅色主题切换
- 分栏拖拽缩放
- Markdown 文件导入
- Markdown 导出
- PDF 导出

## 当前注意事项

项目入口中默认读取根目录下的 `CV.md` 作为初始内容：

```js
import defaultContent from '../CV.md?raw'
```

但当前仓库里还没有这个文件，所以直接执行 `npm run build` 会报错。

如果你希望项目开箱即用，建议在项目根目录补一个 `CV.md` 作为默认简历模板，例如：

```md
# 你的名字

## 个人简介

这里填写你的个人介绍。
```

## 后续可扩展方向

- 增加多套简历主题模板
- 支持头像、页眉、页脚等固定区域
- 支持自动分页和多页 PDF 输出优化
- 支持内容本地持久化
- 支持导出为图片或 HTML

## License

仅作个人学习与项目实践使用。
