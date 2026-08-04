<template>
  <div class="app-shell">
    <nav class="page-tabs" aria-label="页面导航">
      <button
        type="button"
        :class="{ active: currentPage === 'student' }"
        @click="currentPage = 'student'"
      >
        学生管理
      </button>
      <button
        type="button"
        :class="{ active: currentPage === 'order' }"
        @click="currentPage = 'order'"
      >
        简单点菜
      </button>
    </nav>

    <Student v-if="currentPage === 'student'" />
    <Order v-else />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Student from './views/Student.vue'
import Order from './views/Order.vue'

const currentPage = ref<'student' | 'order'>('student')
</script>

<style scoped>
.app-shell {
  min-height: 100vh;
}

.page-tabs {
  position: sticky;
  z-index: 20;
  top: 0;
  display: flex;
  justify-content: center;
  gap: 8px;
  padding: 12px 20px;
  border-bottom: 1px solid #e8ecf3;
  background: rgb(255 255 255 / 92%);
  backdrop-filter: blur(12px);
}

.page-tabs button {
  padding: 9px 18px;
  border: 0;
  border-radius: 999px;
  color: #667085;
  font-weight: 700;
  background: transparent;
  cursor: pointer;
}

.page-tabs button:hover {
  color: #f04438;
  background: #fff1ed;
}

.page-tabs button.active {
  color: white;
  background: #f04438;
}
</style>
