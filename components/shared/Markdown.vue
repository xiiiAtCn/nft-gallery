<template>
  <VueMarkdown
    ref="markdown"
    :source="source"
    :options="highlightOptions"
    class="content-markdown" />
</template>

<script lang="ts" setup>
import VueMarkdown from 'vue-markdown-render'
import hljs from 'highlight.js'
import { useRedirectModal } from '@/components/redirect/useRedirectModal'

const markdown = ref<HTMLElement | null>(null)

defineProps<{
  source: string
}>()

useRedirectModal(markdown)

const highlightOptions = {
  html: true,
  linkify: true,
  typographer: true,
  highlight: (code: string, lang: string) => {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return (
          '<pre class="hljs"><code>' +
          // eslint-disable-next-line @typescript-eslint/no-explicit-any
          hljs.highlight(code, { language: lang } as any).value +
          '</code></pre>'
        )
      } catch (__) {
        return ''
      }
    }
    return hljs.highlightAuto(code).value
  },
}
</script>
