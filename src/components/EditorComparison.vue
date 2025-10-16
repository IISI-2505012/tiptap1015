<template>
  <div class="editor-tutorial">
    <section class="tutorial-section">
      <h2 class="section-title">
        <span class="section-number">1</span>
        實際編輯器體驗
      </h2>

      <div class="editor-demo-grid">
        <div class="editor-demo">
          <h3 class="demo-title ckeditor-color">
            <span class="icon">📝</span>
            CKEditor 5
          </h3>
          <div class="editor-container-wrapper ckeditor-wrapper">
            <div ref="ckeditorContainer" class="editor-container"></div>
          </div>

          <!-- CKEditor 操作按鈕 -->
          <div class="editor-controls">
            <button class="btn btn-warning" @click="addPeriodCKEditor">句尾加表情</button>
            <button class="btn btn-info" @click="showCKEditorOutput">顯示輸出內容</button>
          </div>

          <!-- CKEditor 輸出格式展示 -->
          <div v-if="ckeditorContent" class="output-display">
            <h4>HTML 輸出格式：</h4>
            <pre class="output-content">{{ ckeditorContent }}</pre>
          </div>
        </div>

        <div class="editor-demo">
          <h3 class="demo-title tiptap-color">
            <span class="icon">⚡</span>
            Tiptap
          </h3>
          <div class="editor-container-wrapper tiptap-wrapper">
            <div class="tiptap-editor-container">
              <!-- Tiptap 自訂工具列 -->
              <div class="tiptap-toolbar">
                <button
                  class="toolbar-btn"
                  :class="{ active: tiptapEditor?.isActive('bold') }"
                  @click="tiptapEditor?.chain().focus().toggleBold().run()"
                  :disabled="!tiptapEditor"
                  title="粗體"
                >
                  <strong>B</strong>
                </button>
                <button
                  class="toolbar-btn"
                  :class="{ active: tiptapEditor?.isActive('italic') }"
                  @click="tiptapEditor?.chain().focus().toggleItalic().run()"
                  :disabled="!tiptapEditor"
                  title="斜體"
                >
                  <em>I</em>
                </button>
                <div class="toolbar-divider"></div>
                <button
                  class="toolbar-btn"
                  :class="{ active: tiptapEditor?.isActive('bulletList') }"
                  @click="tiptapEditor?.chain().focus().toggleBulletList().run()"
                  :disabled="!tiptapEditor"
                  title="項目符號"
                >
                  ⬤
                </button>
                <button
                  class="toolbar-btn"
                  :class="{ active: tiptapEditor?.isActive('orderedList') }"
                  @click="tiptapEditor?.chain().focus().toggleOrderedList().run()"
                  :disabled="!tiptapEditor"
                  title="編號清單"
                >
                  1.
                </button>
                <div class="toolbar-divider"></div>
                <button
                  class="toolbar-btn"
                  @click="tiptapEditor?.chain().focus().undo().run()"
                  :disabled="!tiptapEditor?.can().undo()"
                  title="復原"
                >
                  ↶
                </button>
                <button
                  class="toolbar-btn"
                  @click="tiptapEditor?.chain().focus().redo().run()"
                  :disabled="!tiptapEditor?.can().redo()"
                  title="重做"
                >
                  ↷
                </button>
              </div>
              <div ref="tiptapContainer" class="editor-container"></div>
            </div>
          </div>

          <!-- Tiptap 操作按鈕 -->
          <div class="editor-controls">
            <button
              class="btn btn-primary"
              @click="addPeriodTiptap"
              :disabled="!tiptapEditor || tiptapLoading || !!tiptapError"
            >
              句尾加表情
            </button>
            <button
              class="btn btn-success"
              @click="showTiptapOutput"
              :disabled="!tiptapEditor || tiptapLoading || !!tiptapError"
            >
              顯示輸出內容
            </button>
          </div>

          <!-- 輸出格式選擇器 -->
          <div class="format-selector">
            <label for="output-format" class="format-label">選擇輸出格式：</label>
            <select
              id="output-format"
              v-model="outputFormat"
              class="format-select"
              @change="showTiptapOutput"
            >
              <option value="json">原始 JSON</option>
              <option value="html">HTML</option>
              <option value="text">純文字</option>
            </select>
          </div>

          <!-- Tiptap 輸出格式展示 -->
          <div v-if="tiptapContent" class="output-display">
            <h4>{{ getFormatTitle(outputFormat) }}：</h4>
            <pre class="output-content">{{
              typeof tiptapContent === 'string'
                ? tiptapContent
                : JSON.stringify(tiptapContent, null, 2)
            }}</pre>
          </div>
        </div>
      </div>
    </section>

    <!-- 2. 程式碼複雜度比較 -->
    <section class="tutorial-section">
      <h2 class="section-title">
        <span class="section-number">2</span>
        自訂功能程式碼複雜度比較
      </h2>

      <div class="code-comparison">
        <div class="code-example">
          <h3 class="code-title ckeditor-color">CKEditor</h3>
          <div class="code-block">
            <pre><code class="language-javascript">const addPeriodCKEditor = () => {
  if (!ckeditorInstance.value) return

  try {
    const editor = ckeditorInstance.value
    const model = editor.model
    const emoticon = '\\( •̀ω•́ )//'

    // 需要深入理解 CKEditor 的 Model-View 架構
    model.change((writer) => {
      const root = model.document.getRoot()
      const lastChild = root.getChild(root.childCount - 1)
      
      if (lastChild && lastChild.name === 'paragraph') {
        const lastText = lastChild.getChild(lastChild.childCount - 1)
        if (lastText && lastText.data && !lastText.data.endsWith(emoticon)) {
          writer.insertText(emoticon, lastChild, 'end')
        }
      }
    })
  } catch (error) {
    // 需要多層錯誤處理和備用方案
    console.error('CKEditor 操作失敗:', error)
    try {
      const currentData = ckeditorInstance.value.getData()
      const newData = currentData.replace(/&lt;\/p&gt;$/, emoticon + '&lt;/p&gt;')
      ckeditorInstance.value.setData(newData)
    } catch (fallbackError) {
      console.error('備用方案也失敗:', fallbackError)
    }
  }
}</code></pre>
          </div>
          <div class="complexity-indicator">
            <span class="complexity-badge high">複雜度：高</span>
            <span class="lines-count">程式碼行數：~25 行</span>
          </div>
        </div>

        <div class="code-example">
          <h3 class="code-title tiptap-color">Tiptap</h3>
          <div class="code-block">
            <pre><code class="language-javascript">const addPeriodTiptap = () => {
  if (!tiptapEditor.value) return

  // 直觀的 Document 操作
  const { doc } = tiptapEditor.value.state
  const lastNode = doc.lastChild
  const emoticon = '\\( •̀ω•́ )//'

  if (lastNode && lastNode.isTextblock) {
    const lastText = lastNode.textContent
    if (!lastText.endsWith(emoticon)) {
      const pos = doc.content.size - 1
      // 鏈式 API，簡潔直觀
      tiptapEditor.value
        .chain()
        .focus()
        .insertContentAt(pos, emoticon)
        .run()
    }
  }
}</code></pre>
          </div>
          <div class="complexity-indicator">
            <span class="complexity-badge low">複雜度：低</span>
            <span class="lines-count">程式碼行數：~12 行</span>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, shallowRef, nextTick } from 'vue'
// 直接匯入已安裝的 Tiptap 套件
import { Editor } from '@tiptap/core'
import StarterKit from '@tiptap/starter-kit'

// 匯入外部樣式檔案
import './EditorComparison.css'

// 編輯器實例 - 使用 shallowRef 避免深度響應式代理
const ckeditorInstance = shallowRef(null)
const tiptapEditor = shallowRef(null)

// 內容狀態
const ckeditorContent = ref('')
const tiptapContent = ref(null)

// 載入狀態
const ckeditorLoading = ref(true)
const tiptapLoading = ref(true)

// 錯誤狀態
const ckeditorError = ref('')
const tiptapError = ref('')

// DOM 引用
const ckeditorContainer = ref(null)
const tiptapContainer = ref(null)

// 初始內容
const initialContent = '這是一段測試內容，用來展示編輯器的差異。你可以在這裡輸入任何文字進行測試'

// 動態載入 CKEditor
const loadCKEditor = () => {
  return new Promise((resolve, reject) => {
    if (window.ClassicEditor) {
      resolve(window.ClassicEditor)
      return
    }

    const script = document.createElement('script')
    script.src = 'https://cdn.ckeditor.com/ckeditor5/40.0.0/classic/ckeditor.js'
    script.onload = () => {
      if (window.ClassicEditor) {
        resolve(window.ClassicEditor)
      } else {
        reject(new Error('CKEditor failed to load'))
      }
    }
    script.onerror = () => reject(new Error('Failed to load CKEditor script'))
    document.head.appendChild(script)
  })
}

// 初始化 CKEditor
const initCKEditor = async () => {
  try {
    console.log('開始初始化 CKEditor...')
    ckeditorError.value = ''

    const ClassicEditor = await loadCKEditor()
    await nextTick()

    if (!ckeditorContainer.value) {
      throw new Error('CKEditor 容器元素不存在')
    }

    const editor = await ClassicEditor.create(ckeditorContainer.value, {
      toolbar: ['bold', 'italic', '|', 'bulletedList', 'numberedList', '|', 'undo', 'redo'],
      language: 'zh',
    })

    editor.setData(`<p>${initialContent}</p>`)
    ckeditorInstance.value = editor
    ckeditorLoading.value = false

    console.log('CKEditor 初始化成功')
  } catch (error) {
    console.error('CKEditor 初始化失敗:', error)
    ckeditorLoading.value = false
    ckeditorError.value = error.message || '未知錯誤'
  }
}

// 初始化 Tiptap
const initTiptap = async () => {
  try {
    console.log('開始初始化 Tiptap...')
    tiptapError.value = ''
    await nextTick()

    if (!tiptapContainer.value) {
      throw new Error('Tiptap 容器元素不存在')
    }

    const editor = new Editor({
      element: tiptapContainer.value,
      extensions: [StarterKit],
      content: `<p>${initialContent}</p>`,
      editorProps: {
        attributes: {
          style:
            'outline: none; padding: 16px; min-height: 300px; background: white; border: none;',
          class: 'tiptap-content',
        },
      },
    })

    tiptapEditor.value = editor
    tiptapLoading.value = false
    console.log('Tiptap 初始化成功')
  } catch (error) {
    console.error('Tiptap 初始化失敗:', error)
    tiptapLoading.value = false
    tiptapError.value = error.message || '未知錯誤'
  }
}

// 生命週期
onMounted(async () => {
  console.log('元件已掛載，開始初始化編輯器...')
  await nextTick()
  await new Promise((resolve) => setTimeout(resolve, 100))

  const initPromises = [initCKEditor(), initTiptap()]
  const results = await Promise.allSettled(initPromises)

  results.forEach((result, index) => {
    const editorName = index === 0 ? 'CKEditor' : 'Tiptap'
    if (result.status === 'rejected') {
      console.error(`${editorName} 初始化失敗:`, result.reason)
    } else {
      console.log(`${editorName} 初始化成功`)
    }
  })
})

// CKEditor 操作函式
const addPeriodCKEditor = () => {
  if (!ckeditorInstance.value) return

  try {
    const currentData = ckeditorInstance.value.getData()
    const emoticon = '\\( •̀ω•́ )//'

    if (!currentData.includes(emoticon)) {
      const newData = currentData.replace(/(<\/p>)$/i, emoticon + '$1')
      ckeditorInstance.value.setData(newData)
    }
  } catch (error) {
    console.error('CKEditor 操作失敗:', error)
  }
}

const showCKEditorOutput = () => {
  if (ckeditorInstance.value) {
    try {
      ckeditorContent.value = ckeditorInstance.value.getData()
    } catch (error) {
      console.error('取得 CKEditor 內容失敗:', error)
      ckeditorContent.value = '取得內容失敗'
    }
  }
}

// Tiptap 操作函式
const addPeriodTiptap = () => {
  if (!tiptapEditor.value) return

  // 透過 Transaction 精準操作
  const { doc } = tiptapEditor.value.state
  const lastNode = doc.lastChild
  const emoticon = '\\( •̀ω•́ )//'

  if (lastNode && lastNode.isTextblock) {
    const lastText = lastNode.textContent
    if (!lastText.endsWith(emoticon)) {
      const pos = doc.content.size - 1
      tiptapEditor.value.chain().focus().insertContentAt(pos, emoticon).run()
    }
  }
}

// 自訂輸出格式的函式
const formatTiptapOutput = (format) => {
  if (!tiptapEditor.value) return null

  switch (format) {
    case 'json':
      return tiptapEditor.value.getJSON()

    case 'html':
      return tiptapEditor.value.getHTML()

    case 'text':
      return tiptapEditor.value.getText()

    default:
      return tiptapEditor.value.getJSON()
  }
}

// 輸出格式狀態
const outputFormat = ref('json')

// 取得格式標題的函式
const getFormatTitle = (format) => {
  const titles = {
    json: '原始 JSON 格式',
    html: 'HTML 格式',
    text: '純文字格式',
  }
  return titles[format] || 'JSON 格式'
}

const showTiptapOutput = () => {
  if (tiptapEditor.value) {
    tiptapContent.value = formatTiptapOutput(outputFormat.value)
  }
}

onBeforeUnmount(() => {
  if (ckeditorInstance.value) {
    try {
      ckeditorInstance.value.destroy()
      console.log('CKEditor 已銷毀')
    } catch (error) {
      console.error('CKEditor 銷毀失敗:', error)
    } finally {
      ckeditorInstance.value = null
    }
  }

  if (tiptapEditor.value) {
    try {
      tiptapEditor.value.destroy()
      console.log('Tiptap 已銷毀')
    } catch (error) {
      console.error('Tiptap 銷毀失敗:', error)
    } finally {
      tiptapEditor.value = null
    }
  }
})
</script>
