<template>
  <div class="pattern-container">
    <h2>1. 싱글톤 패턴 (Singleton Pattern)</h2>
    <p class="description">
      클래스의 인스턴스가 오직 하나만 존재하도록 보장하고, 이 인스턴스에 대한 전역 접근점을 제공하는
      생성 패턴입니다.
    </p>

    <div class="example-section">
      <h3>"즉시 초기화" (Eager Initialization) 예제</h3>
      <p>애플리케이션 시작 시점에 인스턴스를 미리 생성합니다.</p>
      <button @click="testEagerSingleton" class="test-btn">싱글톤 인스턴스 2개 가져오기</button>
      <div class="output" v-if="eagerOutput">
        <pre>{{ eagerOutput }}</pre>
      </div>
    </div>

    <div class="code-section">
      <h4>코드 예제:</h4>
      <pre><code>{{ codeExample }}</code></pre>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

// --- 싱글톤 클래스 정의 (즉시 초기화 방식) ---

/**
 * Eager Initialization (즉시 초기화) 방식의 싱글톤 클래스입니다.
 * 클래스가 로드되는 시점에 instance가 즉시 생성됩니다.
 */
class Singleton {
  // 1. private static readonly instance: 클래스 로드 시점에 인스턴스를 생성
  private static readonly instance: Singleton = new Singleton()

  private creationTime: string

  // 2. private constructor: 외부에서의 'new' 호출을 막습니다.
  private constructor() {
    this.creationTime = new Date().toLocaleTimeString('ko-KR')
    console.log(`[${this.creationTime}] Eager Singleton 인스턴스 생성됨.`)
  }

  // 3. public static getInstance: 유일한 인스턴스 접근 메서드
  public static getInstance(): Singleton {
    // 이미 생성된 인스턴스를 반환
    return Singleton.instance
  }

  // 예제용 비즈니스 로직
  public operation(): string {
    return `안녕하세요! 저는 ${this.creationTime}에 생성된 싱글톤입니다. 👋`
  }
}

// --- Vue 로직 ---

const eagerOutput = ref<string>('')

const testEagerSingleton = () => {
  // 인스턴스를 두 번 요청합니다.
  const s1 = Singleton.getInstance()
  const s2 = Singleton.getInstance()

  const output = [
    `s1.operation(): ${s1.operation()}`,
    `s2.operation(): ${s2.operation()}`,
    ``,
    `s1 === s2 : ${s1 === s2}`,
  ].join('\n')

  eagerOutput.value = output

  // 개발자 콘솔에서도 확인 (인스턴스 생성 로그는 버튼 클릭 시점이 아닌,
  // 페이지 로드 시점에 한 번만 찍혀야 합니다.)
  console.log('s1과 s2는 동일한가?', s1 === s2)
}

// --- 코드 예시 표시용 ---

const codeExample = `class Singleton {
  // 1. 클래스 로드 시점에 인스턴스를 바로 생성 (즉시 초기화)
  private static readonly instance: Singleton = new Singleton();

  // 2. 외부 생성 방지
  private constructor() {
    // ... 초기화 로직 ...
  }

  // 3. 유일한 접근점 제공
  public static getInstance(): Singleton {
    return Singleton.instance;
  }

  public operation(): void {
    // ...
  }
}

// --- 사용 ---
const s1 = Singleton.getInstance();
const s2 = Singleton.getInstance();

// s1과 s2는 항상 동일한 인스턴스입니다.
console.log(s1 === s2); // true`
</script>

<style scoped>
/* 이전 FactoryMethodPattern에서 사용한 스타일과 동일하게 적용 */
.pattern-container {
  padding: 2rem;
  max-width: 900px;
  margin: 20px auto;
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

h2 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.description {
  color: #666;
  margin-bottom: 2rem;
  line-height: 1.6;
  background-color: #f8f9fa;
  padding: 16px;
  border-radius: 8px;
}

.example-section {
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  margin-bottom: 2rem;
}

.example-section h3 {
  margin-top: 0;
}
.example-section p {
  font-size: 15px;
  color: #333;
}

.button-group {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.test-btn {
  background: #42b983;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 0.5rem; /* p 태그와의 간격 */
}
.test-btn:hover {
  background: #36a473;
}

.output {
  margin-top: 1.5rem;
  background: #2c3e50;
  color: #ecf0f1;
  padding: 1rem;
  border-radius: 4px;
  white-space: pre-wrap;
}

.code-section {
  background: #282c34;
  padding: 1.5rem;
  border-radius: 8px;
  overflow-x: auto;
}

.code-section h4 {
  color: #abb2bf;
  margin-top: 0;
  border-bottom: 1px solid #444;
  padding-bottom: 0.5rem;
}

.code-section pre {
  color: #abb2bf;
  font-family: 'Courier New', monospace;
  font-size: 14px;
}
</style>
