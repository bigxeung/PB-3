<template>
  <div class="pattern-container">
    <h2>4. 어댑터 패턴 (Adapter Pattern)</h2>
    <p class="description">
      호환되지 않는 인터페이스를 가진 클래스들을 함께 작동할 수 있도록 변환해주는 구조
      패턴입니다.<br />
      클라이언트가 기대하는 인터페이스(Target)로 기존 클래스(Adaptee)를 감싸줍니다.
    </p>

    <div class="example-section">
      <h3>데이터 전송 시뮬레이터</h3>
      <p>클라이언트는 <code>sendRequest(data)</code> 메서드로 데이터를 전송하길 원합니다.</p>
      <div class="button-group">
        <button @click="runClientCode" class="test-btn">어댑터를 통해 데이터 전송</button>
      </div>
      <div class="output" v-if="output">
        <pre>{{ output }}</pre>
      </div>
    </div>

    <div class="code-section">
      <h4>코드:</h4>
      <pre><code>{{ codeExample }}</code></pre>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

// --- 1. Adaptee (적응 대상) ---
// 우리가 가지고 있는 기존의 서비스.
// 인터페이스가 클라이언트의 요구와 맞지 않습니다.
class OldAnalyticsService {
  // 클라이언트가 원하는 'sendRequest'가 아닌, 'logEvent' 메서드를 가지고 있음
  public logEvent(eventName: string, eventValue: number): string {
    const message = `[OldAnalytics] 이벤트 기록: ${eventName} (값: ${eventValue})`
    console.log(message)
    return message
  }
}

// --- 2. Target (타겟 인터페이스) ---
// 클라이언트가 기대하는 인터페이스입니다.
interface AnalyticsService {
  sendRequest(data: { type: 'click' | 'purchase'; amount: number }): string
}

// --- 3. Adapter (어댑터) ---
// Target 인터페이스를 구현합니다.
// 내부적으로 Adaptee(OldAnalyticsService)의 인스턴스를 가지고 있습니다.
class AnalyticsAdapter implements AnalyticsService {
  // Adaptee 객체를 내부에 감쌈
  private readonly adaptee: OldAnalyticsService

  constructor(adaptee: OldAnalyticsService) {
    this.adaptee = adaptee
  }

  // Target 인터페이스의 메서드 구현
  public sendRequest(data: { type: 'click' | 'purchase'; amount: number }): string {
    console.log('[Adapter] 클라이언트의 sendRequest() 호출을 받음.')

    // Target의 인터페이스를 Adaptee의 인터페이스로 "변환" 또는 "번역"
    // 'data' 객체를 'eventName'과 'eventValue'로 분해하여 전달
    console.log('[Adapter] OldAnalyticsService의 logEvent()로 변환하여 호출...')

    const result = this.adaptee.logEvent(data.type, data.amount)

    return `[Adapter] 변환 완료:\n${result}`
  }
}

// --- 4. Client (클라이언트) ---
// Target 인터페이스(AnalyticsService)에만 의존하여 코드를 작성합니다.
class Client {
  // 클라이언트는 Target 인터페이스 타입의 객체를 주입받음
  public sendData(service: AnalyticsService) {
    const dataToLog: { type: 'click' | 'purchase'; amount: number } = {
      type: 'purchase',
      amount: 15000
    }

    // 클라이언트는 Target의 메서드(sendRequest)만 알고 호출
    return service.sendRequest(dataToLog)
  }
}

// --- Vue 로직 ---

const output = ref<string>('')

const runClientCode = () => {
  // 1. 기존의 서비스(Adaptee) 생성
  const oldService = new OldAnalyticsService()

  // 2. 어댑터 생성 (기존 서비스를 감쌈)
  // 클라이언트는 oldService를 직접 사용할 수 없음 (인터페이스 불일치)
  const adapter = new AnalyticsAdapter(oldService)

  // 3. 클라이언트 생성
  const client = new Client()

  // 4. 클라이언트에게 어댑터를 (Target 인터페이스인 척) 주입
  // 클라이언트는 자신이 어댑터를 사용하는지 모름
  output.value = client.sendData(adapter)
}

const codeExample = `// 1. Adaptee (기존 클래스)
class OldAnalyticsService {
  // 인터페이스가 다름 (logEvent)
  public logEvent(eventName: string, eventValue: number): string {
    /* ... */
  }
}

// 2. Target (클라이언트가 원하는 인터페이스)
interface AnalyticsService {
  sendRequest(data: { type: string; amount: number }): string;
}

// 3. Adapter (변환기)
// Target 인터페이스를 구현하고, Adaptee를 내부에 감싼다.
class AnalyticsAdapter implements AnalyticsService {
  private readonly adaptee: OldAnalyticsService;

  constructor(adaptee: OldAnalyticsService) {
    this.adaptee = adaptee;
  }

  // Target의 메서드를 구현
  public sendRequest(data: { type: string; amount: number }): string {
    // 클라이언트의 호출(data)을 Adaptee의 메서드(eventName, eventValue)로 변환
    return this.adaptee.logEvent(data.type, data.amount);
  }
}

// --- 사용 ---
const oldService = new OldAnalyticsService();
// oldService를 어댑터로 감싼다
const adapter = new AnalyticsAdapter(oldService);

// 클라이언트는 Target 인터페이스(adapter)를 통해
// 실제로는 OldService를 사용하게 된다.
const client = new Client();
client.sendData(adapter);`
</script>

<style scoped>
/* 어댑터 패턴 - 코랄/오렌지 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #fff5e6 0%, #ffe8d1 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(250, 112, 154, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🧡';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #ff6b6b;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(255, 107, 107, 0.1);
}

.description {
  color: #ee5a6f;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #fa709a;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(250, 112, 154, 0.1);
}

.example-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(250, 112, 154, 0.2);
  box-shadow: 0 8px 25px rgba(250, 112, 154, 0.1);
}

.example-section h3 {
  margin-top: 0;
  color: #ff7979;
  font-size: 1.5rem;
  font-weight: 700;
}
.example-section p {
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

.test-btn {
  background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 6px 20px rgba(250, 112, 154, 0.3);
  transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  margin-top: 0.5rem;
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(250, 112, 154, 0.4);
}
.test-btn:active {
  transform: translateY(0) scale(0.98);
}

.output {
  margin-top: 1.5rem;
  background: linear-gradient(135deg, #ff6348 0%, #ff9a56 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(255, 99, 72, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
}

.code-section {
  background: linear-gradient(135deg, #1e1e2e 0%, #2a2a3e 100%);
  padding: 2rem;
  border-radius: 25px;
  overflow-x: auto;
  border: 3px solid rgba(250, 112, 154, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #fa709a;
  margin-top: 0;
  border-bottom: 2px solid #fa709a;
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
