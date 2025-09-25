<template>
  <div class="p-6 max-w-2xl mx-auto">
    <div class="flex items-center gap-2 mb-4">
      <input
        v-model="keyword"
        placeholder="Enter keywords (space separated)"
        class="flex-1 px-3 py-2 border rounded-md"
      />
      <button @click="keyword = ''" class="px-3 py-2 bg-gray-200 rounded">Clear</button>
    </div>

    <div class="text-sm text-gray-500 mb-4">
      Showing <strong>{{ filteredList.length }}</strong> / {{ dataList.length }} results
    </div>

    <ul class="space-y-2">
      <li
        v-for="(text, index) in filteredList"
        :key="index"
        v-html="highlightText(text)"
        class="p-3 rounded bg-gray-50"
      />
    </ul>
  </div>
</template>

<script setup lang="ts">
import { findAll } from "highlight-words-core";
import { computed, ref } from "vue";

const keyword = ref("");
const dataList = ref<string[]>([
  "This is some text to highlight.",
  "Another example with Highlight core.",
  "Vue3 + Tailwind + Element Plus integration.",
  "Searching keywords inside long content works too.",
  "Highlight multiple words like Vue Tailwind core."
]);

// 将输入按空格分词（支持多个关键词）
const searchWords = computed(() =>
  keyword.value
    .trim()
    .split(/\s+/)
    .filter(Boolean)
);

// 先进行筛选：只保留包含任一关键词的条目
const filteredList = computed(() => {
  const words = searchWords.value;
  if (!words.length) return dataList.value;
  return dataList.value.filter((text) => {
    const chunks = findAll({
      searchWords: words,
      textToHighlight: text,
      autoEscape: true, // 把正则特殊字符当普通字符处理
    });
    return chunks.some((c) => c.highlight);
  });
});

// 简单的 HTML 转义，防止用户输入导致 XSS
function escapeHtml(s: string) {
  return s
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

// 使用 highlight-words-core 返回的 chunks 拼接高亮 HTML
function highlightText(text: string) {
  const words = searchWords.value;
  if (!words.length) return escapeHtml(text);

  const chunks = findAll({
    searchWords: words,
    textToHighlight: text,
    autoEscape: true,
  });

  return chunks
    .map((chunk) => {
      const part = escapeHtml(text.slice(chunk.start, chunk.end));
      if (chunk.highlight) {
        // 这里用了 Tailwind 的样式类（黄底＋深蓝字），你可以改成任意类名
        return `<mark class="bg-yellow-200 text-blue-800 px-0.5 rounded">${part}</mark>`;
      }
      return part;
    })
    .join("");
}
</script>

<style scoped>
/* mark 的默认样式被浏览器处理了，上面使用 Tailwind 类已足够 */
</style>
