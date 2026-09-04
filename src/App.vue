<script setup lang="ts">
import HelloWorld from "@/components/HelloWorld.vue";
</script>

<template>
  <!-- 根容器承担配色与 color-scheme:light-dark() 需要祖先带 scheme-* 才会按系统深浅色切换 -->
  <main
    class="flex min-h-screen items-center justify-center bg-background p-6 font-sans text-foreground scheme-light-dark"
  >
    <HelloWorld />
  </main>
</template>
