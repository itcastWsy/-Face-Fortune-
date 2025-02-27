<script setup>
import FaceFortune from './components/FaceFortune.vue'
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>面相算命</h1>
      <p>通过人工智能分析您的面相特征，揭示您的性格、事业和婚姻运势</p>
    </header>
    <main>
      <FaceFortune />
    </main>
    <footer class="app-footer">
      <p>© 2025 面相算命 - 仅供娱乐，请勿当真</p>
    </footer>
  </div>
</template>

<style>
:root {
  --primary-color: #409eff;
  --success-color: #67c23a;
  --warning-color: #e6a23c;
  --danger-color: #f56c6c;
  --info-color: #909399;
  --text-color: #303133;
  --border-color: #dcdfe6;
  --background-color: #f5f7fa;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--text-color);
  background-color: var(--background-color);
  min-height: 100vh;
}

.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.app-header {
  text-align: center;
  padding: 20px 0;
  background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
  color: white;
  margin-bottom: 20px;
}

.app-header h1 {
  margin: 0;
  font-size: 2.5rem;
  font-weight: bold;
}

.app-header p {
  margin: 10px 0 0;
  font-size: 1rem;
  opacity: 0.9;
}

main {
  flex: 1;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.app-footer {
  text-align: center;
  padding: 20px;
  margin-top: 40px;
  font-size: 0.9rem;
  color: var(--info-color);
  border-top: 1px solid var(--border-color);
}

@media (max-width: 768px) {
  .app-header h1 {
    font-size: 2rem;
  }

  .app-header p {
    font-size: 0.9rem;
  }
}
</style>
