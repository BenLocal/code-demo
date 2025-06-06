<template>
  <div>
    <codemirror v-model="code" placeholder="Code goes here..." :style="{ height: '400px' }" :autofocus="true"
      :indent-with-tab="true" :tab-size="2" :extensions="extensions" @ready="handleReady" @change="handleChange" />
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, shallowRef } from 'vue'
import { Codemirror } from 'vue-codemirror'
import { javascript } from '@codemirror/lang-javascript'
import { oneDark } from '@codemirror/theme-one-dark'

const code = ref(`console.log('Hello, world!')`)
const extensions = shallowRef([javascript(), oneDark])

onMounted(async () => {
  try {
    // 检查服务器是否可用
    const testWs = new WebSocket('ws://localhost:3000/javascript')

    testWs.onopen = async () => {
      console.log('Language server is available, connecting...')
      testWs.close()

      // 如果服务器可用，加载语言服务器
      const { languageServer } = await import('codemirror-languageserver')

      const serverUri = "ws://localhost:3000/javascript"
      const workspaceUri = 'file:///tmp/aaa/'
      const filename = 'main.js'

      const ls = languageServer({
        serverUri,
        rootUri: workspaceUri,
        workspaceFolders: [{
          name: 'My Project',
          uri: workspaceUri
        }],
        documentUri: `${workspaceUri}${filename}`,
        languageId: 'javascript',
      })

      // 更新扩展列表
      extensions.value = [...extensions.value, ls]
      console.log('Language server connected successfully')
    }

    testWs.onerror = () => {
      console.warn('Language server not available, continuing without LSP features')
      testWs.close()
    }

  } catch (error) {
    console.warn('Failed to connect to language server:', error)
  }
})

// Codemirror EditorView instance ref
const view = shallowRef()
const handleReady = (payload: {
  view: import("@codemirror/view").EditorView;
}) => {
  view.value = payload.view
}

// Status is available at all times via Codemirror EditorView
const getCodemirrorStates = () => {
  // const state = view.value.state
  // const ranges = state.selection.ranges
  // const selected = ranges.reduce((r, range) => r + range.to - range.from, 0)
  // console.log('Selected characters:', selected)
  // const cursor = ranges[0].anchor
  // console.log('Cursor position:', cursor)
  // const length = state.doc.length
  // console.log('Document length:', length)
  // const lines = state.doc.lines
  // console.log('Number of lines:', lines)
}

const handleChange = (payload: string) => {
  console.log('Code changed:', payload)
  getCodemirrorStates()
}
</script>

