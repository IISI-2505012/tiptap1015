<template>
  <div class="editor-tutorial">
    <!-- <div class="header">
      <h1>💻 CKEditor vs Tiptap 實作比較</h1>
      <p>程式碼複雜度、資料結構與實際應用展示</p>
    </div> -->

    <!-- 1. 實際編輯器展示 -->
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
            <div v-if="ckeditorLoading && !ckeditorError" class="loading-indicator">
              <div class="spinner"></div>
              <p>載入 CKEditor 中...</p>
            </div>
            <div v-if="ckeditorError" class="error-indicator">
              <div class="error-icon">❌</div>
              <p><strong>CKEditor 載入失敗</strong></p>
              <small>{{ ckeditorError }}</small>
            </div>
            <div
              v-show="!ckeditorLoading && !ckeditorError"
              ref="ckeditorContainer"
              class="editor-container"
            ></div>
          </div>

          <!-- CKEditor 操作按鈕 -->
          <div class="editor-controls">
            <button
              class="btn btn-warning"
              @click="addPeriodCKEditor"
              :disabled="!ckeditorInstance || ckeditorLoading || !!ckeditorError"
            >
              一鍵加表情
            </button>
            <button
              class="btn btn-info"
              @click="showCKEditorOutput"
              :disabled="!ckeditorInstance || ckeditorLoading || !!ckeditorError"
            >
              顯示輸出內容
            </button>
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
            <div v-if="tiptapLoading && !tiptapError" class="loading-indicator">
              <div class="spinner"></div>
              <p>載入 Tiptap 中...</p>
            </div>
            <div v-if="tiptapError" class="error-indicator">
              <div class="error-icon">❌</div>
              <p><strong>Tiptap 載入失敗</strong></p>
              <small>{{ tiptapError }}</small>
            </div>
            <div v-show="!tiptapLoading && !tiptapError" class="tiptap-editor-container">
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
              一鍵加表情
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
              <option value="markdown">Markdown</option>
              <option value="custom">簡化 JSON</option>
              <option value="flat">扁平化格式</option>
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
    // 使用更可靠的方式操作 CKEditor
    const currentData = ckeditorInstance.value.getData()
    const emoticon = '\\( •̀ω•́ )//'

    if (!currentData.includes(emoticon)) {
      // 使用正則表達式來處理各種可能的結尾格式
      const newData = currentData.replace(/(<\/p>)$/i, emoticon + '$1')
      ckeditorInstance.value.setData(newData)
    }
  } catch (error) {
    console.error('CKEditor 操作失敗:', error)
    // 備用方案：直接在編輯器中插入內容
    try {
      const editor = ckeditorInstance.value
      editor.model.change((writer) => {
        const insertPosition = editor.model.document.selection.getFirstPosition()
        writer.insertText(emoticon, insertPosition)
      })
    } catch (fallbackError) {
      console.error('CKEditor 備用方案也失敗:', fallbackError)
    }
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

    case 'markdown':
      // 簡單的 Markdown 轉換
      const json = tiptapEditor.value.getJSON()
      return jsonToMarkdown(json)

    case 'custom':
      // 自訂的簡化格式
      const customJson = tiptapEditor.value.getJSON()
      return simplifyJsonStructure(customJson)

    case 'flat':
      // 扁平化的內容格式
      return flattenContent(tiptapEditor.value.getJSON())

    default:
      return tiptapEditor.value.getJSON()
  }
}

// JSON 轉 Markdown 的輔助函式
const jsonToMarkdown = (json) => {
  if (!json.content) return ''

  return json.content
    .map((node) => {
      switch (node.type) {
        case 'paragraph':
          const text = node.content?.map((textNode) => textNode.text || '').join('') || ''
          return text
        case 'bulletList':
          return (
            node.content
              ?.map(
                (item) =>
                  `- ${item.content?.[0]?.content?.map((textNode) => textNode.text || '').join('') || ''}`
              )
              .join('\n') || ''
          )
        case 'orderedList':
          return (
            node.content
              ?.map(
                (item, index) =>
                  `${index + 1}. ${item.content?.[0]?.content?.map((textNode) => textNode.text || '').join('') || ''}`
              )
              .join('\n') || ''
          )
        default:
          return ''
      }
    })
    .join('\n\n')
}

// 簡化 JSON 結構的輔助函式
const simplifyJsonStructure = (json) => {
  if (!json.content) return { content: '' }

  const simplified = {
    type: 'document',
    content: json.content.map((node) => {
      switch (node.type) {
        case 'paragraph':
          return {
            type: 'paragraph',
            text: node.content?.map((textNode) => textNode.text || '').join('') || '',
          }
        case 'bulletList':
          return {
            type: 'list',
            listType: 'bullet',
            items:
              node.content?.map(
                (item) =>
                  item.content?.[0]?.content?.map((textNode) => textNode.text || '').join('') || ''
              ) || [],
          }
        case 'orderedList':
          return {
            type: 'list',
            listType: 'ordered',
            items:
              node.content?.map(
                (item) =>
                  item.content?.[0]?.content?.map((textNode) => textNode.text || '').join('') || ''
              ) || [],
          }
        default:
          return node
      }
    }),
  }

  return simplified
}

// 扁平化內容的輔助函式
const flattenContent = (json) => {
  if (!json.content) return []

  const flattened = []

  json.content.forEach((node) => {
    switch (node.type) {
      case 'paragraph':
        const text = node.content?.map((textNode) => textNode.text || '').join('') || ''
        if (text.trim()) {
          flattened.push({
            type: 'text',
            content: text,
          })
        }
        break
      case 'bulletList':
      case 'orderedList':
        node.content?.forEach((item, index) => {
          const itemText =
            item.content?.[0]?.content?.map((textNode) => textNode.text || '').join('') || ''
          if (itemText.trim()) {
            flattened.push({
              type: 'listItem',
              listType: node.type === 'bulletList' ? 'bullet' : 'ordered',
              index: node.type === 'orderedList' ? index + 1 : null,
              content: itemText,
            })
          }
        })
        break
    }
  })

  return flattened
}

// 輸出格式狀態
const outputFormat = ref('json')

// 取得格式標題的函式
const getFormatTitle = (format) => {
  const titles = {
    json: '原始 JSON 格式',
    html: 'HTML 格式',
    text: '純文字格式',
    markdown: 'Markdown 格式',
    custom: '簡化 JSON 格式',
    flat: '扁平化格式',
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

<style scoped>
.editor-tutorial {
  min-height: 100vh;
  width: 100%;
  padding: 20px;
  box-sizing: border-box;
  font-family: 'Microsoft JhengHei', 'Segoe UI', sans-serif;
  line-height: 1.6;
  justify-content: center;
}

.header {
  text-align: center;
  margin-bottom: 40px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.header h1 {
  margin: 0 0 10px 0;
  font-size: 2.2em;
  font-weight: 700;
}

.header p {
  margin: 0;
  font-size: 1.1em;
  opacity: 0.9;
}

/* 教學章節樣式 */
.tutorial-section {
  background: white;
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid #e1e5e9;
  min-height: calc(100vh - 60px);
  width: 100%;
  box-sizing: border-box;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 30px;
  font-size: 1.8em;
  font-weight: 700;
  color: #202124;
  border-bottom: 3px solid #f0f0f0;
  padding-bottom: 15px;
}

.section-number {
  background: #585d7598;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 0.8em;
}

/* 編輯器展示 */
.editor-demo-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
  min-height: calc(100vh - 200px);
  align-items: start;
  width: 100%;
}

.editor-demo {
  border-radius: 12px;
  border: 2px solid;
  overflow: visible;
  display: flex;
  flex-direction: column;
  min-height: 100%;
  width: 100%;
  min-width: 0;
}

.editor-demo:nth-child(1) {
  border-color: #ff6b35;
}

.editor-demo:nth-child(2) {
  border-color: #4285f4;
}

.demo-title {
  margin: 0;
  padding: 15px 20px;
  font-size: 1.1em;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
}

.ckeditor-color {
  background-color: #fff5f2;
  color: #ff6b35;
  border-left: 4px solid #ff6b35;
}

.tiptap-color {
  background-color: #f0f4ff;
  color: #4285f4;
  border-left: 4px solid #4285f4;
}

.editor-container-wrapper {
  border-bottom: 1px solid #e1e5e9;
  flex: 0 0 auto;
  display: flex;
  flex-direction: column;
  width: 100%;
  min-width: 0;
  min-height: 400px;
}

.ckeditor-wrapper {
  background: #fff5f2;
}

.tiptap-wrapper {
  background: #f0f4ff;
}

.editor-container {
  /* min-height: 300px; */
  width: 100%;
  /* flex: 1; */
  display: flex;
  flex-direction: column;
}

.demo-description {
  padding: 15px 20px;
  margin: 0;
  font-size: 0.9em;
  color: #5f6368;
  background: #fafbfc;
}

.icon {
  font-size: 1.2em;
}

/* Tiptap 編輯器容器 */
.tiptap-editor-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
}

/* Tiptap 工具列樣式 */
.tiptap-toolbar {
  display: flex;
  align-items: center;
  gap: 2px;
  padding: 8px 12px;
  background: white;
  border-bottom: 1px solid #e1e5e9;
  flex-shrink: 0;
}

.toolbar-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: 1px solid #e1e5e9;
  background: white;
  color: #5f6368;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.2s ease;
}

.toolbar-btn:hover:not(:disabled) {
  background: #f8f9fa;
  border-color: #dadce0;
}

.toolbar-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.toolbar-btn.active {
  background: #4285f4;
  color: white;
  border-color: #4285f4;
}

.toolbar-divider {
  width: 1px;
  height: 24px;
  background: #e1e5e9;
  margin: 0 4px;
}

/* Tiptap 內容樣式 */
:deep(.tiptap-content) {
  flex: 1;
  overflow-y: auto;
}

:deep(.tiptap-content p) {
  margin: 1em 0;
}

:deep(.tiptap-content p:first-child) {
  margin-top: 0;
}

:deep(.tiptap-content p:last-child) {
  margin-bottom: 0;
}

:deep(.tiptap-content ul, .tiptap-content ol) {
  padding-left: 1.5em;
  margin: 1em 0;
}

:deep(.tiptap-content li) {
  margin: 0.25em 0;
}

/* 編輯器控制按鈕 */
.editor-controls {
  padding: 15px 20px;
  background: #f8f9fa;
  border-top: 1px solid #e1e5e9;
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

/* 格式選擇器樣式 */
.format-selector {
  padding: 15px 20px;
  background: #f8f9fa;
  border-top: 1px solid #e1e5e9;
  display: flex;
  align-items: center;
  gap: 10px;
}

.format-label {
  font-size: 14px;
  font-weight: 600;
  color: #495057;
  margin: 0;
}

.format-select {
  padding: 6px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  background: white;
  font-size: 14px;
  color: #495057;
  min-width: 150px;
  cursor: pointer;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
}

.format-select:focus {
  border-color: #4285f4;
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(66, 133, 244, 0.25);
}

.format-select:hover {
  border-color: #adb5bd;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
  min-width: 100px;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary {
  background: #4285f4;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #3367d6;
  transform: translateY(-1px);
}

.btn-warning {
  background: #ff6b35;
  color: white;
}

.btn-warning:hover:not(:disabled) {
  background: #e55a2b;
  transform: translateY(-1px);
}

.btn-info {
  background: #17a2b8;
  color: white;
}

.btn-info:hover:not(:disabled) {
  background: #138496;
  transform: translateY(-1px);
}

.btn-success {
  background: #28a745;
  color: white;
}

.btn-success:hover:not(:disabled) {
  background: #218838;
  transform: translateY(-1px);
}

/* 輸出格式展示 */
.output-display {
  padding: 15px 20px;
  background: #f8f9fa;
  border-top: 1px solid #e1e5e9;
}

.output-display h4 {
  margin: 0 0 10px 0;
  color: #495057;
  font-size: 0.9em;
  font-weight: 600;
}

.output-content {
  margin: 0;
  padding: 12px;
  background: #1e1e1e;
  color: #d4d4d4;
  border-radius: 6px;
  font-size: 11px;
  line-height: 1.4;
  white-space: pre-wrap;
  word-wrap: break-word;
  max-height: 200px;
  overflow-y: auto;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
}

/* 程式碼比較樣式 */
.code-comparison {
  display: flex;
  flex-direction: row;
  gap: 30px;
}

.code-example {
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #e1e5e9;
}

.code-title {
  margin: 0;
  padding: 15px 20px;
  font-size: 1.2em;
  font-weight: 600;
  border-bottom: 1px solid #e1e5e9;
}

.code-block {
  background: #1e1e1e;
  color: #d4d4d4;
  padding: 20px;
  overflow-x: auto;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.5;
}

.code-block pre {
  margin: 0;
}

.complexity-indicator {
  padding: 15px 20px;
  background: #f8f9fa;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #e1e5e9;
}

.complexity-badge {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
}

.complexity-badge.high {
  background: #ffeaea;
  color: #d73027;
}

.complexity-badge.low {
  background: #e8f5e8;
  color: #2e7d32;
}

.lines-count {
  font-size: 12px;
  color: #666;
}

/* 資料結構比較 */
.data-structure-comparison {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.structure-example {
  border-radius: 12px;
  border: 1px solid #e1e5e9;
  overflow: hidden;
}

.structure-content {
  padding: 20px;
}

.data-format {
  margin-bottom: 25px;
}

.data-format h4 {
  margin: 0 0 10px 0;
  color: #202124;
  font-size: 1em;
}

.data-format pre {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 6px;
  font-size: 12px;
  overflow-x: auto;
  border-left: 3px solid #e1e5e9;
}

.operation-style h4 {
  margin: 0 0 10px 0;
  color: #202124;
  font-size: 1em;
}

.operation-style ul {
  margin: 0;
  padding-left: 20px;
}

.operation-style li {
  margin-bottom: 8px;
  color: #5f6368;
}

/* 載入指示器樣式 */
.loading-indicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  color: #666;
}

.spinner {
  width: 32px;
  height: 32px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #4285f4;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 12px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.loading-indicator p {
  margin: 0;
  font-size: 14px;
  color: #666;
}

/* 錯誤指示器樣式 */
.error-indicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  color: #c00;
  background: #fee;
  border-radius: 6px;
  margin: 8px;
}

.error-icon {
  font-size: 24px;
  margin-bottom: 12px;
}

.error-indicator p {
  margin: 0 0 8px 0;
  font-size: 14px;
  font-weight: 600;
}

.error-indicator small {
  font-size: 12px;
  color: #666;
  text-align: center;
  font-family: monospace;
}

/* CKEditor 客製化樣式 */
:deep(.ck-editor) {
  height: 100% !important;
  display: flex !important;
  flex-direction: column !important;
}

:deep(.ck-editor__main) {
  flex: 1 !important;
  height: 100% !important;
  border-radius: 0 0 8px 8px !important;
}

:deep(.ck-editor__editable) {
  min-height: 300px !important;
  height: 100% !important;
  padding: 16px !important;
  border: none !important;
}

:deep(.ck-toolbar) {
  border-radius: 8px 8px 0 0 !important;
  border-bottom: 1px solid #e1e5e9 !important;
  flex-shrink: 0 !important;
}
</style>
