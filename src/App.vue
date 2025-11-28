<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

// 存储从后端获取的资源列表
const resources = ref([]);
const loading = ref(true);
const error = ref(null);

// 异步函数：从后端获取数据
async function fetchResources() {
  loading.value = true;
  error.value = null;
  try {
    // 关键点：使用相对路径 /api/resources
    // Pages 代理将确保这个请求被转发到您的 ECS
    const response = await axios.get('/api/resources'); 
    resources.value = response.data;
  } catch (err) {
    console.error("API Call Failed:", err);
    error.value = "无法连接到后端 API，请检查 Pages 代理配置和 ECS 状态。";
  } finally {
    loading.value = false;
  }
}

// 组件加载完成后立即调用获取数据
onMounted(() => {
  fetchResources();
});
</script>

<template>
  <header>
    <h1>资源站项目</h1>
  </header>

  <main>
    <h2>资源列表</h2>
    
    <div v-if="loading">
      <p>正在加载资源...</p>
    </div>

    <div v-else-if="error">
      <p style="color: red;">错误: {{ error }}</p>
    </div>

    <div v-else>
      <ul v-if="resources.length > 0">
        <li v-for="resource in resources" :key="resource.id">
          <strong>{{ resource.name }}</strong> (ID: {{ resource.id }})
          <p>{{ resource.description }}</p>
          <a :href="'/api/download/' + resource.fileKey">下载 ({{ resource.size }} bytes)</a>
        </li>
      </ul>
      <p v-else>后端未返回任何资源。</p>
    </div>
  </main>
</template>

<style scoped>
/* 样式保持简洁，可以根据需要自定义 */
header {
  line-height: 1.5;
}
main {
  margin-top: 20px;
}
</style>