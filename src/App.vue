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
import { shallowRef, type Component } from 'vue'

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
const patterns: { key: string; name: string; component: Component }[] = [
  // 생성
  { key: 'singleton', name: '1. Singleton', component: shallowRef(SingletonEx) },
  { key: 'factoryMethod', name: '2. Factory Method', component: shallowRef(FactoryMethodEx) },
  {
    key: 'abstractFactory',
    name: '3. Abstract Factory',
    component: shallowRef(AbstractFactoryEx),
  },
  // 구조
  { key: 'adapter', name: '4. Adapter', component: shallowRef(AdaptorEx) },
  { key: 'decorator', name: '5. Decorator', component: shallowRef(DecoratorEx) },
  { key: 'proxy', name: '6. Proxy', component: shallowRef(ProxyEx) },
  // 행위
  { key: 'observer', name: '7. Observer', component: shallowRef(ObserverEx) },
  { key: 'visitor', name: '8. Visitor', component: shallowRef(VisitorEx) },
  { key: 'state', name: '9. State', component: shallowRef(StateEx) },
]

// 현재 선택된 패턴 컴포넌트 (기본값: 싱글톤)
// Vue 3에서 동적 컴포넌트는 성능을 위해 'shallowRef'를 사용하는 것이 권장됩니다.
const currentPattern = shallowRef(SingletonEx)
</script>

<style>
/* 전역 스타일 (scoped가 아님)
  App.vue는 최상위 컴포넌트이므로, 여기에 전역 폰트 및 배경색을 정의합니다.
*/
body {
  margin: 0;
  font-family:
    -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f4f7f6; /* 앱 전체 배경색 */
}
</style>

<style scoped>
#app-container {
  color: #2c3e50;
  min-height: 100vh;
}

.app-header {
  background: white;
  padding: 1.5rem 2rem;
  border-bottom: 1px solid #e0e0e0;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.02);
}

.app-header h1 {
  margin: 0;
  font-size: 2rem;
  color: #42b983; /* Vue 색상 */
}

.app-header p {
  font-size: 1.1rem;
  color: #666;
  margin-top: 0.5rem;
  margin-bottom: 0;
}

/* 탭 네비게이션 바 */
.pattern-tabs {
  display: flex;
  flex-wrap: wrap; /* 탭이 많으면 줄바꿈 */
  justify-content: center;
  background-color: #ffffff;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #e0e0e0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);

  /* 스크롤 시 상단에 고정 */
  position: sticky;
  top: 0;
  z-index: 10;
}

.pattern-tabs button {
  padding: 0.6rem 1rem;
  margin: 0.25rem;
  border: 1px solid #dcdcdc;
  background: #fdfdfd;
  color: #34495e;
  font-size: 14px;
  font-weight: 500;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.pattern-tabs button:hover {
  background: #f0f0f0;
  border-color: #b0b0b0;
}

/* 활성화된 탭 버튼 */
.pattern-tabs button.active {
  background: #42b983;
  color: white;
  border-color: #42b983;
  box-shadow: 0 2px 5px rgba(66, 185, 131, 0.3);
}

/* 각 패턴의 컴포넌트가 표시될 메인 영역 */
.pattern-content {
  /*
    각 패턴 컴포넌트(.pattern-container)가
    최대 900px ~ 1100px의 max-width를 가지고 있으므로
    여기서 별도 너비 지정 없이 패딩만 줍니다.
  */
  padding: 1rem; /* 모바일 대응 */
}

/* 태블릿 및 데스크탑 */
@media (min-width: 768px) {
  .app-header {
    padding: 2rem 3rem;
  }
  .app-header h1 {
    font-size: 2.5rem;
  }
  .app-header p {
    font-size: 1.2rem;
  }
  .pattern-tabs {
    padding: 1rem 1.5rem;
  }
  .pattern-tabs button {
    padding: 0.75rem 1.25rem;
  }
  .pattern-content {
    padding: 2rem;
  }
}
</style>
