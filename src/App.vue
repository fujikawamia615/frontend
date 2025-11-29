<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

// ===== 登录状态管理 =====
const username = ref('');
const password = ref('');
const loginError = ref('');
const isLoggedIn = ref(false);
const showSearchView = ref(false);
const searchQuery = ref('');

// 固定账号密码（仅用于本地测试）
const VALID_USERNAME = 'xixixi';
const VALID_PASSWORD = '123456';

function handleLogin() {
  if (username.value === VALID_USERNAME && password.value === VALID_PASSWORD) {
    loginError.value = '';
    isLoggedIn.value = true;
    fetchResources(); // 登录成功后加载资源
  } else {
    loginError.value = '用户名或密码错误！';
  }
}

function logout() {
  isLoggedIn.value = false;
  username.value = '';
  password.value = '';
  resources.value = [];
  showSearchView.value = false;
}

// ===== 资源数据 =====
const resources = ref([]);
const loading = ref(false);
const error = ref(null);

// 使用绝对路径（按主人要求）
const API_BASE = 'http://39.105.154.74:8080';

async function fetchResources() {
  loading.value = true;
  error.value = null;
  try {
    const response = await axios.get(`${API_BASE}/api/resources`);
    resources.value = response.data;
  } catch (err) {
    console.error('请求失败:', err);
    error.value = '无法连接到后端，请检查网络或服务器是否运行。';
  } finally {
    loading.value = false;
  }
}

// ===== 工具函数 =====
function formatSize(bytes) {
  if (!bytes || bytes <= 0) return '0 B';
  const k = 1024;
  const sizes = ['B', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}

// ===== 页面加载 =====
onMounted(() => {
  // 页面加载动画
  document.body.classList.add('loaded');
});

// ===== 搜索功能 =====
const searchResults = ref([]);
const isSearching = ref(false);

async function performSearch() {
  if (!searchQuery.value.trim()) {
    searchResults.value = [];
    isSearching.value = false;
    return;
  }
  
  isSearching.value = true;
  try {
    const response = await axios.get(`${API_BASE}/api/resources`);
    // 模拟搜索 - 在实际应用中，这里应该是后端API
    const allResources = response.data;
    searchResults.value = allResources.filter(resource => 
      resource.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      (resource.description && resource.description.toLowerCase().includes(searchQuery.value.toLowerCase()))
    );
  } catch (err) {
    console.error('搜索失败:', err);
    searchResults.value = [];
  } finally {
    isSearching.value = false;
  }
}

function toggleSearchView() {
  showSearchView.value = !showSearchView.value;
  if (!showSearchView.value) {
    searchQuery.value = '';
    searchResults.value = [];
  }
}

function closeSearchView() {
  showSearchView.value = false;
  searchQuery.value = '';
  searchResults.value = [];
}
</script>

<template>
  <!-- 登录界面（ASMR风格） -->
  <div v-if="!isLoggedIn" class="login-wrapper">
    <div class="login-box">
      <div class="login-header">
        <h1 class="title">✨ ASMR音声站</h1>
        <p class="subtitle">发现更多精彩音声内容</p>
      </div>
      
      <form @submit.prevent="handleLogin" class="login-form">
        <div class="input-group">
          <input
            v-model="username"
            type="text"
            placeholder="用户名（xixixi）"
            class="input"
            required
          />
        </div>
        <div class="input-group">
          <input
            v-model="password"
            type="password"
            placeholder="密码（123456）"
            class="input"
            required
          />
        </div>
        <button type="submit" class="login-btn">登录</button>
        <p v-if="loginError" class="error">{{ loginError }}</p>
      </form>
      
      <div class="login-footer">
        <p>欢迎来到ASMR世界</p>
      </div>
    </div>
  </div>

  <!-- ✨ 主界面：ASMR风格 ✨ -->
  <div v-else class="main-layout">
    <!-- 搜索视图 -->
    <div v-if="showSearchView" class="search-view">
      <header class="search-header">
        <div class="search-header-content">
          <button @click="closeSearchView" class="back-btn">←</button>
          <div class="search-bar">
            <input
              v-model="searchQuery"
              @input="performSearch"
              type="text"
              placeholder="搜索ASMR资源..."
              class="search-input"
              autofocus
            >
            <button v-if="searchQuery" @click="searchQuery = ''; searchResults = []" class="clear-btn">×</button>
          </div>
        </div>
      </header>
      
      <div class="search-results">
        <!-- 搜索中 -->
        <div v-if="isSearching" class="loading-state">
          <div class="loading-spinner"></div>
          <p>搜索中...</p>
        </div>
        
        <!-- 搜索结果 -->
        <div v-else-if="searchResults.length > 0" class="resource-grid">
          <div
            v-for="resource in searchResults"
            :key="resource.id"
            class="resource-card"
          >
            <div class="card-content">
              <h3 class="card-title">{{ resource.name }}</h3>
              <p class="card-desc">{{ resource.description || '暂无描述' }}</p>
              <div class="card-meta">
                <span class="file-type">{{ resource.fileType || '其他' }}</span>
                <span class="size">{{ formatSize(resource.size) }}</span>
              </div>
              <div class="card-actions">
                <a
                  :href="`${API_BASE}/api/download/${resource.fileKey}`"
                  target="_blank"
                  class="download-btn"
                >
                  📥 下载
                </a>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 无搜索结果 -->
        <div v-else-if="searchQuery && !isSearching" class="no-results">
          <p>没有找到相关资源</p>
        </div>
        
        <!-- 搜索历史/热门搜索（当没有搜索词时显示） -->
        <div v-else class="search-suggestions">
          <h3 class="suggestions-title">热门搜索</h3>
          <div class="suggestion-tags">
            <span class="suggestion-tag" @click="searchQuery = 'ASMR'; performSearch()">ASMR</span>
            <span class="suggestion-tag" @click="searchQuery = '轻音乐'; performSearch()">轻音乐</span>
            <span class="suggestion-tag" @click="searchQuery = '白噪音'; performSearch()">白噪音</span>
            <span class="suggestion-tag" @click="searchQuery = '自然音'; performSearch()">自然音</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 正常视图 -->
    <div v-else class="normal-view">
      <!-- 顶部导航 -->
      <header class="site-header">
        <div class="header-content">
          <div class="logo-area">
            <h1>✨ ASMR音声站</h1>
            <span class="tagline">发现更多精彩音声内容</span>
          </div>
          <nav class="main-nav">
            <a href="#" class="nav-link">首页</a>
            <a href="#" class="nav-link">ASMR</a>
            <a href="#" class="nav-link">轻音乐</a>
            <a href="#" class="nav-link">白噪音</a>
            <a href="#" class="nav-link">自然音</a>
            <a href="#" class="nav-link">排行榜</a>
          </nav>
          <div class="header-actions">
            <button @click="toggleSearchView" class="search-icon">
              🔍
            </button>
            <button @click="logout" class="logout-btn">退出登录</button>
          </div>
        </div>
      </header>

      <div class="container">
        <!-- 分类导航 -->
        <div class="category-nav">
          <a href="#" class="category-link active">全部</a>
          <a href="#" class="category-link">ASMR</a>
          <a href="#" class="category-link">轻音乐</a>
          <a href="#" class="category-link">白噪音</a>
          <a href="#" class="category-link">自然音</a>
          <a href="#" class="category-link">冥想</a>
          <a href="#" class="category-link">睡眠</a>
          <a href="#" class="category-link">放松</a>
        </div>

        <!-- 热门推荐 -->
        <div class="section-header">
          <h2 class="section-title">热门推荐</h2>
        </div>
        <div class="resource-grid">
          <!-- 加载中 -->
          <div v-if="loading" class="loading-state">
            <div class="loading-spinner"></div>
            <p>正在加载资源...</p>
          </div>

          <!-- 错误提示 -->
          <div v-else-if="error" class="error-state">
            <p class="error-text">{{ error }}</p>
            <button @click="fetchResources" class="retry-btn">重新加载</button>
          </div>

          <!-- 资源卡片列表 -->
          <div
            v-for="resource in resources.slice(0, 6)"
            :key="resource.id"
            class="resource-card"
          >
            <div class="card-content">
              <h3 class="card-title">{{ resource.name }}</h3>
              <p class="card-desc">{{ resource.description || '暂无描述' }}</p>
              <div class="card-meta">
                <span class="file-type">{{ resource.fileType || '其他' }}</span>
                <span class="size">{{ formatSize(resource.size) }}</span>
              </div>
              <div class="card-actions">
                <a
                  :href="`${API_BASE}/api/download/${resource.fileKey}`"
                  target="_blank"
                  class="download-btn"
                >
                  📥 下载
                </a>
              </div>
            </div>
          </div>
        </div>

        <!-- 最新资源 -->
        <div class="section-header">
          <h2 class="section-title">最新资源</h2>
          <a href="#" class="view-more">查看更多</a>
        </div>
        <div class="resource-grid">
          <!-- 资源卡片列表 -->
          <div
            v-for="resource in resources"
            :key="resource.id"
            class="resource-card"
          >
            <div class="card-content">
              <h3 class="card-title">{{ resource.name }}</h3>
              <p class="card-desc">{{ resource.description || '暂无描述' }}</p>
              <div class="card-meta">
                <span class="file-type">{{ resource.fileType || '其他' }}</span>
                <span class="size">{{ formatSize(resource.size) }}</span>
              </div>
              <div class="card-actions">
                <a
                  :href="`${API_BASE}/api/download/${resource.fileKey}`"
                  target="_blank"
                  class="download-btn"
                >
                  📥 下载
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* ===== 全局重置 ===== */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  /* ASMR风格渐变背景 */
  background:
    radial-gradient(circle at 10% 20%, rgba(106, 90, 249, 0.1) 0%, transparent 20%),
    radial-gradient(circle at 90% 80%, rgba(255, 107, 203, 0.1) 0%, transparent 20%),
    linear-gradient(135deg, #f8f9ff 0%, #fefaff 100%);
  min-height: 100vh;
  font-family: 'Microsoft YaHei', 'PingFang SC', -apple-system, BlinkMacSystemFont, sans-serif;
  color: #333;
  line-height: 1.6;
  transition: all 0.3s ease;
}

body.loaded {
  background:
    radial-gradient(circle at 10% 20%, rgba(106, 90, 249, 0.15) 0%, transparent 25%),
    radial-gradient(circle at 90% 80%, rgba(255, 107, 203, 0.15) 0%, transparent 25%),
    linear-gradient(135deg, #f8f9ff 0%, #fefaff 100%);
}

/* ===== 登录页 ===== */
.login-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  background: transparent;
}

.login-box {
  width: 100%;
  max-width: 420px;
  padding: 40px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.1),
    0 0 0 1px rgba(255, 255, 255, 0.2);
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.login-header {
  margin-bottom: 32px;
}

.title {
  font-size: 32px;
  font-weight: 700;
  margin-bottom: 8px;
  background: linear-gradient(90deg, #6a5af9, #ff6bcb);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  letter-spacing: -0.5px;
}

.subtitle {
  color: #666;
  font-size: 16px;
  margin-bottom: 24px;
}

.login-form {
  margin-bottom: 24px;
}

.input-group {
  margin-bottom: 16px;
}

.input {
  width: 100%;
  padding: 14px 18px;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  font-size: 16px;
  background: rgba(255, 255, 255, 0.8);
  transition: all 0.3s ease;
}

.input:focus {
  outline: none;
  border-color: #6a5af9;
  box-shadow: 0 0 0 3px rgba(106, 90, 249, 0.1);
  background: white;
}

.login-btn {
  width: 100%;
  padding: 14px;
  background: linear-gradient(90deg, #6a5af9, #8a7bff);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-bottom: 16px;
}

.login-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(106, 90, 249, 0.3);
}

.error {
  color: #ef4444;
  font-size: 14px;
  margin-top: 12px;
  padding: 8px 12px;
  background: rgba(239, 68, 68, 0.1);
  border-radius: 6px;
  border: 1px solid rgba(239, 68, 68, 0.2);
}

.login-footer {
  padding-top: 20px;
  border-top: 1px solid #e2e8f0;
  margin-top: 20px;
}

.login-footer p {
  color: #666;
  font-size: 14px;
}

/* ===== 主界面 ===== */
.main-layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 正常视图 */
.normal-view {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.site-header {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.3);
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 70px;
}

.logo-area {
  display: flex;
  flex-direction: column;
}

.logo-area h1 {
  font-size: 24px;
  background: linear-gradient(90deg, #6a5af9, #ff6bcb);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  font-weight: 700;
  margin: 0;
}

.tagline {
  font-size: 12px;
  color: #666;
  margin-top: 2px;
}

.main-nav {
  display: flex;
  gap: 24px;
}

.nav-link {
  color: #666;
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: color 0.3s ease;
  padding: 8px 12px;
  border-radius: 6px;
}

.nav-link:hover,
.nav-link.active {
  color: #6a5af9;
  background: rgba(106, 90, 249, 0.1);
}

.header-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}

.search-icon {
  width: 40px;
  height: 40px;
  border: none;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.search-icon:hover {
  background: rgba(106, 90, 249, 0.1);
  color: #6a5af9;
}

.logout-btn {
  background: linear-gradient(90deg, #6a5af9, #8a7bff);
  color: white;
  border: none;
  padding: 8px 20px;
  border-radius: 20px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.logout-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 15px rgba(106, 90, 249, 0.3);
}

.container {
  max-width: 1200px;
  margin: 24px auto;
  padding: 0 24px;
  flex: 1;
}

/* 分类导航 */
.category-nav {
  display: flex;
  gap: 12px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.category-link {
  padding: 6px 16px;
  background: rgba(255, 255, 255, 0.8);
  color: #666;
  text-decoration: none;
  border-radius: 20px;
  font-size: 14px;
  transition: all 0.3s ease;
  border: 1px solid #e2e8f0;
}

.category-link:hover,
.category-link.active {
  background: linear-gradient(90deg, #6a5af9, #8a7bff);
  color: white;
  border-color: transparent;
}

/* 章节标题 */
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  padding: 0 8px;
}

.section-title {
  font-size: 24px;
  font-weight: 700;
  color: #222;
  background: linear-gradient(90deg, #6a5af9, #ff6bcb);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

.view-more {
  color: #6a5af9;
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: color 0.3s ease;
}

.view-more:hover {
  color: #8a7bff;
}

.resource-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
  margin-bottom: 40px;
}

/* 资源卡片 */
.resource-card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 
    0 10px 30px rgba(0, 0, 0, 0.1),
    0 0 0 1px rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.3);
  height: fit-content;
  display: flex;
  flex-direction: column;
}

.resource-card:hover {
  transform: translateY(-4px);
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.15),
    0 0 0 1px rgba(106, 90, 249, 0.2);
}

.card-content {
  padding: 16px;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 8px;
  color: #222;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-desc {
  font-size: 13px;
  color: #666;
  margin-bottom: 12px;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  flex: 1;
}

.card-meta {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
  font-size: 12px;
}

.file-type {
  background: #eef2ff;
  color: #4f46e5;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;
}

.size {
  color: #666;
  font-size: 12px;
}

.card-actions {
  display: flex;
  gap: 12px;
}

.download-btn {
  flex: 1;
  background: linear-gradient(90deg, #6a5af9, #8a7bff);
  color: white;
  text-decoration: none;
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.3s ease;
  text-align: center;
  border: none;
  cursor: pointer;
}

.download-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 15px rgba(106, 90, 249, 0.3);
}

/* 搜索视图 */
.search-view {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: white;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  backdrop-filter: blur(10px);
}

.search-header {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.3);
  padding: 0 16px;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
}

.search-header-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  height: 70px;
  gap: 12px;
}

.back-btn {
  width: 40px;
  height: 40px;
  border: none;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.back-btn:hover {
  background: rgba(106, 90, 249, 0.1);
  color: #6a5af9;
}

.search-bar {
  flex: 1;
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20px;
  padding: 0 16px;
  border: 1px solid #e2e8f0;
}

.search-input {
  flex: 1;
  border: none;
  background: transparent;
  padding: 12px 0;
  font-size: 16px;
  outline: none;
}

.clear-btn {
  width: 30px;
  height: 30px;
  border: none;
  background: transparent;
  font-size: 18px;
  cursor: pointer;
  border-radius: 50%;
  transition: background 0.3s ease;
}

.clear-btn:hover {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
}

.search-results {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* 搜索建议 */
.search-suggestions {
  padding: 20px 0;
}

.suggestions-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 16px;
  color: #222;
}

.suggestion-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.suggestion-tag {
  padding: 8px 16px;
  background: #e2e8f0;
  color: #4a5568;
  border-radius: 20px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.suggestion-tag:hover {
  background: #cbd5e0;
  color: #2d3748;
}

/* 无结果状态 */
.no-results {
  text-align: center;
  padding: 60px 0;
  color: #666;
}

/* 状态页面 */
.loading-state {
  text-align: center;
  padding: 60px 0;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #e2e8f0;
  border-top: 3px solid #6a5af9;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-state {
  grid-column: 1 / -1;
  text-align: center;
  padding: 60px 0;
}

.error-text {
  color: #ef4444;
  margin-bottom: 16px;
}

.retry-btn {
  background: linear-gradient(90deg, #6a5af9, #8a7bff);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.retry-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 15px rgba(106, 90, 249, 0.3);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header-content {
    padding: 0 16px;
    height: 60px;
    flex-direction: column;
    gap: 12px;
  }
  
  .main-nav {
    gap: 12px;
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .header-actions {
    gap: 8px;
  }
  
  .search-icon {
    width: 36px;
    height: 36px;
    font-size: 16px;
  }
  
  .logo-area h1 {
    font-size: 20px;
  }
  
  .container {
    padding: 0 16px;
    margin: 16px auto;
  }
  
  .section-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }
  
  .section-title {
    font-size: 20px;
  }
  
  .resource-grid {
    grid-template-columns: 1fr;
    gap: 16px;
  }
  
  .category-nav {
    justify-content: center;
  }
  
  .login-box {
    padding: 30px;
    margin: 20px;
  }
  
  .title {
    font-size: 28px;
  }
  
  /* 搜索视图响应式 */
  .search-header-content {
    padding: 0 12px;
    height: 60px;
  }
  
  .search-results {
    padding: 16px;
  }
}
</style>



