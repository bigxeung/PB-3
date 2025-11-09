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

    <div class="comparison-section">
      <h3>📊 비교: 올바른 방법 vs 잘못된 방법</h3>
      <p>싱글톤 패턴을 사용했을 때와 사용하지 않았을 때의 차이를 확인해보세요.</p>

      <div class="button-group">
        <button @click="showGoodExample" class="good-btn">✅ 올바른 방법 (싱글톤 사용)</button>
        <button @click="showBadExample" class="bad-btn">❌ 잘못된 방법 (싱글톤 미사용)</button>
      </div>

      <div class="output good-output" v-if="goodOutput">
        <div class="output-header">✅ 올바른 방법: 싱글톤 패턴 사용</div>
        <pre>{{ goodOutput }}</pre>
        <div class="explanation">
          💡 <strong>장점:</strong> 항상 같은 인스턴스를 사용하므로 메모리 효율적이고, 전역 상태를 일관되게 유지할 수 있습니다.
        </div>
      </div>

      <div class="output bad-output" v-if="badOutput">
        <div class="output-header">❌ 잘못된 방법: 매번 새 인스턴스 생성</div>
        <pre>{{ badOutput }}</pre>
        <div class="explanation">
          ⚠️ <strong>문제점:</strong> 매번 새로운 인스턴스가 생성되어 메모리 낭비가 발생하고, 각 인스턴스의 상태가 달라 데이터 불일치가 발생할 수 있습니다.
        </div>
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

// --- 잘못된 방법: 싱글톤 미사용 클래스 ---
class RegularClass {
  private creationTime: string
  private id: number

  // public constructor: 누구나 new로 생성 가능
  constructor() {
    this.creationTime = new Date().toLocaleTimeString('ko-KR')
    this.id = Math.floor(Math.random() * 10000)
    console.log(`[${this.creationTime}] RegularClass 인스턴스 ${this.id} 생성됨.`)
  }

  public operation(): string {
    return `안녕하세요! 저는 ${this.creationTime}에 생성된 인스턴스 #${this.id}입니다.`
  }
}

// --- Vue 로직 ---

const eagerOutput = ref<string>('')
const goodOutput = ref<string>('')
const badOutput = ref<string>('')

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

const showGoodExample = () => {
  badOutput.value = '' // 다른 출력 숨기기

  const instance1 = Singleton.getInstance()
  const instance2 = Singleton.getInstance()
  const instance3 = Singleton.getInstance()

  const output = [
    `첫 번째 요청: ${instance1.operation()}`,
    `두 번째 요청: ${instance2.operation()}`,
    `세 번째 요청: ${instance3.operation()}`,
    ``,
    `instance1 === instance2: ${instance1 === instance2}`,
    `instance2 === instance3: ${instance2 === instance3}`,
    `instance1 === instance3: ${instance1 === instance3}`,
    ``,
    `✅ 결과: 모두 같은 인스턴스입니다!`,
    `💾 메모리: 인스턴스 1개만 사용`,
  ].join('\n')

  goodOutput.value = output
}

const showBadExample = () => {
  goodOutput.value = '' // 다른 출력 숨기기

  const instance1 = new RegularClass()
  const instance2 = new RegularClass()
  const instance3 = new RegularClass()

  const output = [
    `첫 번째 생성: ${instance1.operation()}`,
    `두 번째 생성: ${instance2.operation()}`,
    `세 번째 생성: ${instance3.operation()}`,
    ``,
    `instance1 === instance2: ${instance1 === instance2}`,
    `instance2 === instance3: ${instance2 === instance3}`,
    `instance1 === instance3: ${instance1 === instance3}`,
    ``,
    `❌ 결과: 모두 다른 인스턴스입니다!`,
    `💾 메모리: 인스턴스 3개 생성 (낭비)`,
    `⚠️  각 인스턴스가 독립적인 상태를 가져 데이터 불일치 발생 가능`,
  ].join('\n')

  badOutput.value = output
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
/* 싱글톤 패턴 - 핑크 그라디언트 테마 🌸 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #fff5f7 0%, #ffe4e9 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(245, 87, 108, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🎀';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #f5576c;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(245, 87, 108, 0.1);
}

.description {
  color: #d946ef;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #f093fb;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(240, 147, 251, 0.1);
}

.example-section,
.comparison-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(245, 87, 108, 0.2);
  box-shadow: 0 8px 25px rgba(245, 87, 108, 0.1);
}

.example-section h3,
.comparison-section h3 {
  margin-top: 0;
  color: #f093fb;
  font-size: 1.5rem;
  font-weight: 700;
}
.example-section p,
.comparison-section p {
  font-size: 16px;
  color: #555;
  line-height: 1.7;
}

.button-group {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
  flex-wrap: wrap;
}

.test-btn,
.good-btn,
.bad-btn {
  border: none;
  padding: 1rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  margin-top: 0.5rem;
}

.test-btn {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
  box-shadow: 0 6px 20px rgba(245, 87, 108, 0.3);
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(245, 87, 108, 0.4);
}
.test-btn:active {
  transform: translateY(0) scale(0.98);
}

.good-btn {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
  color: white;
  box-shadow: 0 6px 20px rgba(67, 233, 123, 0.3);
}
.good-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(67, 233, 123, 0.4);
}

.bad-btn {
  background: linear-gradient(135deg, #ff6b6b 0%, #ff8e53 100%);
  color: white;
  box-shadow: 0 6px 20px rgba(255, 107, 107, 0.3);
}
.bad-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(255, 107, 107, 0.4);
}

.output {
  margin-top: 1.5rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
}

.good-output {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
  border: 3px solid rgba(67, 233, 123, 0.3);
  box-shadow: 0 8px 25px rgba(67, 233, 123, 0.3);
}

.bad-output {
  background: linear-gradient(135deg, #ff6b6b 0%, #ff8e53 100%);
  border: 3px solid rgba(255, 107, 107, 0.3);
  box-shadow: 0 8px 25px rgba(255, 107, 107, 0.3);
}

.output-header {
  font-size: 1.1rem;
  font-weight: 800;
  margin-bottom: 1rem;
  padding-bottom: 0.8rem;
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
}

.explanation {
  margin-top: 1.2rem;
  padding-top: 1.2rem;
  border-top: 2px solid rgba(255, 255, 255, 0.3);
  font-size: 14px;
  line-height: 1.7;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.code-section {
  background: linear-gradient(135deg, #1e1e2e 0%, #2a2a3e 100%);
  padding: 2rem;
  border-radius: 25px;
  overflow-x: auto;
  border: 3px solid rgba(240, 147, 251, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #f093fb;
  margin-top: 0;
  border-bottom: 2px solid #f093fb;
  padding-bottom: 0.8rem;
  font-size: 1.3rem;
  font-weight: 700;
}

.code-section pre {
  color: #e0e0e0;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 15px;
  line-height: 1.7;
  margin: 0;
}
</style>
