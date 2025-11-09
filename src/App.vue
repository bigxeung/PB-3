<template>
  <div id="app-container">
    <header class="app-header">
      <h1>Vue 3 디자인 패턴 예제</h1>
      <p>9가지 기본 디자인 패턴을 Vue 3와 TypeScript로 구현한 예제입니다.</p>
    </header>

    <nav class="pattern-tabs">
      <button
        v-for="pattern in patterns"
        :key="pattern.key"
        :class="{ active: currentPattern === pattern.component }"
        @click="currentPattern = pattern.component"
      >
        {{ pattern.name }}
      </button>
    </nav>

    <main class="pattern-content">
      <!-- 현재 선택된 패턴 컴포넌트가 여기에 렌더링됩니다. -->
      <component :is="currentPattern" />
    </main>
  </div>
</template>

<script setup lang="ts">
import { shallowRef, type Component, markRaw } from 'vue'

// 1. 생성 패턴 (Creational)
import SingletonEx from './patterns/singleton-ex.vue'
import FactoryMethodEx from './patterns/factoryMethod-ex.vue'
import AbstractFactoryEx from './patterns/abstractFactory-ex.vue'

// 2. 구조 패턴 (Structural)
import AdaptorEx from './patterns/adaptor-ex.vue'
import DecoratorEx from './patterns/decorator-ex.vue'
import ProxyEx from './patterns/proxy-ex.vue'

// 3. 행위 패턴 (Behavioral)
import ObserverEx from './patterns/observer-ex.vue'
import VisitorEx from './patterns/visitor-ex.vue'
import StateEx from './patterns/state-ex.vue'

// 네비게이션 및 동적 컴포넌트 목록
const patterns = [
  // 생성
  { key: 'singleton', name: '1. Singleton', component: markRaw(SingletonEx) },
  { key: 'factoryMethod', name: '2. Factory Method', component: markRaw(FactoryMethodEx) },
  {
    key: 'abstractFactory',
    name: '3. Abstract Factory',
    component: markRaw(AbstractFactoryEx),
  },
  // 구조
  { key: 'adapter', name: '4. Adapter', component: markRaw(AdaptorEx) },
  { key: 'decorator', name: '5. Decorator', component: markRaw(DecoratorEx) },
  { key: 'proxy', name: '6. Proxy', component: markRaw(ProxyEx) },
  // 행위
  { key: 'observer', name: '7. Observer', component: markRaw(ObserverEx) },
  { key: 'visitor', name: '8. Visitor', component: markRaw(VisitorEx) },
  { key: 'state', name: '9. State', component: markRaw(StateEx) },
] as const

// 현재 선택된 패턴 컴포넌트 (기본값: 싱글톤)
// Vue 3에서 동적 컴포넌트는 성능을 위해 'shallowRef'를 사용하는 것이 권장됩니다.
const currentPattern = shallowRef<Component>(SingletonEx)
</script>

<style>
/* 전역 스타일 (scoped가 아님) - 귀여운 테마! */
body {
  margin: 0;
  font-family:
    'Segoe UI', 'Apple Color Emoji', 'Segoe UI Emoji', Roboto, 'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* 파스텔 그라디언트 배경 */
  background: linear-gradient(135deg, #ffeef8 0%, #e0f4ff 25%, #fff5e6 50%, #f0e6ff 75%, #e6fff0 100%);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}
</style>

<style scoped>
#app-container {
  color: #2c3e50;
  min-height: 100vh;
}

.app-header {
  background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 50%, #ffecd2 100%);
  padding: 2rem 2rem;
  text-align: center;
  box-shadow: 0 8px 24px rgba(255, 105, 180, 0.2);
  border-radius: 0 0 30px 30px;
  margin-bottom: 1.5rem;
}

.app-header h1 {
  margin: 0;
  font-size: 2.5rem;
  color: #ff1493;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(255, 255, 255, 0.5);
  letter-spacing: 1px;
}

.app-header p {
  font-size: 1.2rem;
  color: #d946ef;
  margin-top: 0.8rem;
  margin-bottom: 0;
  font-weight: 600;
}

/* 탭 네비게이션 바 - 알록달록! */
.pattern-tabs {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.8rem;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  padding: 1.2rem 1.5rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border-radius: 25px;
  margin: 0 1rem 1.5rem 1rem;

  position: sticky;
  top: 10px;
  z-index: 10;
}

.pattern-tabs button {
  padding: 0.8rem 1.3rem;
  margin: 0;
  border: 3px solid transparent;
  background: linear-gradient(white, white) padding-box,
              linear-gradient(135deg, #667eea 0%, #764ba2 100%) border-box;
  color: #5b21b6;
  font-size: 14px;
  font-weight: 700;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  box-shadow: 0 4px 10px rgba(102, 126, 234, 0.15);
}

.pattern-tabs button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

/* 알록달록한 탭 색상 */
.pattern-tabs button:nth-child(1).active { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); border-color: #f5576c; color: white; }
.pattern-tabs button:nth-child(2).active { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); border-color: #00f2fe; color: white; }
.pattern-tabs button:nth-child(3).active { background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); border-color: #38f9d7; color: white; }
.pattern-tabs button:nth-child(4).active { background: linear-gradient(135deg, #fa709a 0%, #fee140 100%); border-color: #fee140; color: white; }
.pattern-tabs button:nth-child(5).active { background: linear-gradient(135deg, #30cfd0 0%, #330867 100%); border-color: #30cfd0; color: white; }
.pattern-tabs button:nth-child(6).active { background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%); border-color: #fed6e3; color: #d946ef; }
.pattern-tabs button:nth-child(7).active { background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%); border-color: #fad0c4; color: #d946ef; }
.pattern-tabs button:nth-child(8).active { background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%); border-color: #fcb69f; color: white; }
.pattern-tabs button:nth-child(9).active { background: linear-gradient(135deg, #e0c3fc 0%, #8ec5fc 100%); border-color: #8ec5fc; color: white; }

.pattern-tabs button.active {
  transform: scale(1.1);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
}

/* 각 패턴의 컴포넌트가 표시될 메인 영역 */
.pattern-content {
  padding: 1rem;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 태블릿 및 데스크탑 */
@media (min-width: 768px) {
  .app-header {
    padding: 2.5rem 3rem;
    border-radius: 0 0 40px 40px;
  }
  .app-header h1 {
    font-size: 3rem;
  }
  .app-header p {
    font-size: 1.3rem;
  }
  .pattern-tabs {
    padding: 1.5rem 2rem;
    margin: 0 2rem 2rem 2rem;
    border-radius: 30px;
  }
  .pattern-tabs button {
    padding: 0.9rem 1.5rem;
    font-size: 15px;
  }
  .pattern-content {
    padding: 2rem;
  }
}
</style>
