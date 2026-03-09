<template>
  <div class="more-content">
    <div class="runtime-container">
      <div class="runtime-card">
        <div class="runtime-title">
          <i class="icon-clock"></i>
          <span>网站运行时间</span>
        </div>
        
        <div class="runtime-info">
          <div class="runtime-item">
            <div class="runtime-value">{{ years }}</div>
            <div class="runtime-label">年</div>
          </div>
          <div class="runtime-item">
            <div class="runtime-value">{{ months }}</div>
            <div class="runtime-label">月</div>
          </div>
          <div class="runtime-item">
            <div class="runtime-value">{{ days }}</div>
            <div class="runtime-label">天</div>
          </div>
          <div class="runtime-item">
            <div class="runtime-value">{{ hours }}</div>
            <div class="runtime-label">时</div>
          </div>
          <div class="runtime-item">
            <div class="runtime-value">{{ minutes }}</div>
            <div class="runtime-label">分</div>
          </div>
          <div class="runtime-item">
            <div class="runtime-value">{{ seconds }}</div>
            <div class="runtime-label">秒</div>
          </div>
        </div>
        
        <div class="date-display">
          <div class="start-date">建站时间: {{ startDate }}</div>
          <div class="current-date">当前时间: {{ currentDate }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

// 设置您的建站日期（格式：YYYY-MM-DD）
const siteCreationDate = "2025-08-24";

const years = ref(0);
const months = ref(0);
const days = ref(0);
const hours = ref(0);
const minutes = ref(0);
const seconds = ref(0);
const startDate = ref("");
const currentDate = ref("");
let timer = null;

// 计算运行时间
const calculateRuntime = () => {
  const now = new Date();
  const start = new Date(siteCreationDate);
  
  // 计算时间差（毫秒）
  const diffTime = Math.abs(now - start);
  
  // 计算年、月、日、小时、分钟、秒
  years.value = Math.floor(diffTime / (1000 * 60 * 60 * 24 * 365.25));
  const remainingAfterYears = diffTime % (1000 * 60 * 60 * 24 * 365.25);
  
  months.value = Math.floor(remainingAfterYears / (1000 * 60 * 60 * 24 * 30.44));
  const remainingAfterMonths = remainingAfterYears % (1000 * 60 * 60 * 24 * 30.44);
  
  days.value = Math.floor(remainingAfterMonths / (1000 * 60 * 60 * 24));
  const remainingAfterDays = remainingAfterMonths % (1000 * 60 * 60 * 24);
  
  hours.value = Math.floor(remainingAfterDays / (1000 * 60 * 60));
  const remainingAfterHours = remainingAfterDays % (1000 * 60 * 60);
  
  minutes.value = Math.floor(remainingAfterHours / (1000 * 60));
  const remainingAfterMinutes = remainingAfterHours % (1000 * 60);
  
  seconds.value = Math.floor(remainingAfterMinutes / 1000);
  
  // 格式化日期显示
  const formatDate = (date) => {
    return `${date.getFullYear()}年${(date.getMonth() + 1).toString().padStart(2, '0')}月${date.getDate().toString().padStart(2, '0')}日 ${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}:${date.getSeconds().toString().padStart(2, '0')}`;
  };
  
  startDate.value = formatDate(start);
  currentDate.value = formatDate(now);
};

// 组件挂载时启动定时器
onMounted(() => {
  calculateRuntime(); // 立即计算一次
  timer = setInterval(calculateRuntime, 1000); // 每秒更新一次
});

// 组件卸载时清除定时器
onUnmounted(() => {
  if (timer) {
    clearInterval(timer);
  }
});
</script>

<style lang="scss" scoped>
.more-content {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
  width: 100%;
  height: 100%;
  padding: 20px;
}

.runtime-container {
  width: 100%;
  max-width: 800px;
}

.runtime-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 30px;
  backdrop-filter: blur(10px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  text-align: center;
}

.runtime-title {
  font-size: 1.8rem;
  margin-bottom: 25px;
  color: #42b883;
  display: flex;
  align-items: center;
  justify-content: center;
  
  .icon-clock {
    display: inline-block;
    width: 30px;
    height: 30px;
    margin-right: 12px;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%2342b883'%3E%3Cpath d='M12 2C6.486 2 2 6.486 2 12s4.486 10 10 10 10-4.486 10-10S17.514 2 12 2zm0 18c-4.411 0-8-3.589-8-8s3.589-8 8-8 8 3.589 8 8-3.589 8-8 8z'/%3E%3Cpath d='M13 7h-2v6h6v-2h-4z'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: center;
  }
}

.runtime-info {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  margin-bottom: 25px;
  
  @media (min-width: 768px) {
    grid-template-columns: repeat(6, 1fr);
  }
}

.runtime-item {
  background: rgba(0, 0, 0, 0.2);
  padding: 15px 10px;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.runtime-value {
  font-size: 1.8rem;
  font-weight: bold;
  color: #42b883;
  margin-bottom: 5px;
  font-variant-numeric: tabular-nums;
}

.runtime-label {
  font-size: 0.9rem;
  opacity: 0.9;
}

.date-display {
  background: rgba(0, 0, 0, 0.2);
  padding: 15px;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.start-date, .current-date {
  font-size: 1rem;
  opacity: 0.9;
}

// 响应式设计
@media (max-width: 600px) {
  .runtime-title {
    font-size: 1.5rem;
    
    .icon-clock {
      width: 25px;
      height: 25px;
    }
  }
  
  .runtime-value {
    font-size: 1.5rem;
  }
  
  .runtime-info {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>