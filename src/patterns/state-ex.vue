<template>
  <div class="pattern-container">
    <h2>9. 상태 패턴 (State Pattern)</h2>
    <p class="description">
      객체의 내부 상태가 변경됨에 따라, 객체의 행동을 변경할 수 있게 해주는 행위 패턴입니다.<br />
      객체는 마치 자신의 클래스를 변경하는 것처럼 보입니다. (행동 로직을 상태 객체로 분리)
    </p>

    <div class="example-section">
      <h3>신호등 시뮬레이터</h3>
      <p>
        '다음 상태로 변경' 버튼을 누르면, 신호등(Context)의 현재 상태(State) 객체가<br />
        자신의 행동을 수행한 뒤, 다음 상태로 스스로 전이시킵니다.
      </p>
      <div class="button-group">
        <button @click="changeState" class="test-btn">다음 상태로 변경</button>
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
import { ref, onMounted } from 'vue'

// --- 1. Context (문맥) ---
// 현재 상태(State) 객체를 유지하며, 클라이언트의 요청을 현재 상태에게 위임합니다.
// (인터페이스 대신 구체 클래스로 바로 정의)
class TrafficLightContext {
  // 현재 상태를 참조
  private currentState: TrafficLightState

  // 초기 상태를 RedState로 설정
  constructor() {
    this.currentState = new RedState()
    console.log(`[Context] 신호등 초기 상태: ${this.currentState.constructor.name}`)
  }

  // 상태를 변경하는 메서드 (State 객체가 이 메서드를 호출하여 Context의 상태를 바꿈)
  public setState(newState: TrafficLightState): void {
    this.currentState = newState
    console.log(`[Context] 상태 전이됨 -> ${this.currentState.constructor.name}`)
  }

  // 클라이언트가 호출하는 메서드
  // 실제 행동은 현재 상태(currentState)에게 위임
  public request(): string {
    return this.currentState.handle(this)
  }
}

// --- 2. State (상태 인터페이스) ---
// 특정 상태의 행동(handle)을 정의합니다.
// 이 행동은 Context를 다음 상태로 전이시킬 수 있습니다.
interface TrafficLightState {
  handle(context: TrafficLightContext): string
}

// --- 3. ConcreteState (구체적인 상태) ---
// State 인터페이스를 구현합니다.

// 빨간불 상태
class RedState implements TrafficLightState {
  public handle(context: TrafficLightContext): string {
    const action = '🔴 정지 (STOP)'

    // 행동을 수행한 뒤, 다음 상태(GreenState)로 Context를 전이시킴
    context.setState(new GreenState())

    return action
  }
}

// 초록불 상태
class GreenState implements TrafficLightState {
  public handle(context: TrafficLightContext): string {
    const action = '🟢 진행 (GO)'

    // 다음 상태(YellowState)로 전이
    context.setState(new YellowState())

    return action
  }
}

// 노란불 상태
class YellowState implements TrafficLightState {
  public handle(context: TrafficLightContext): string {
    const action = '🟡 주의 (CAUTION)'

    // 다음 상태(RedState)로 전이 (순환)
    context.setState(new RedState())

    return action
  }
}

// --- Vue 로직 ---

const output = ref<string>('')
let trafficLight: TrafficLightContext

onMounted(() => {
  // Context(신호등) 인스턴스 생성. 초기 상태는 RedState.
  trafficLight = new TrafficLightContext()
  output.value = '--- 신호등(Context) 생성됨 (초기 상태: Red) ---'
})

const changeState = () => {
  // 클라이언트는 그저 Context의 request()만 호출
  // 실제 행동과 상태 전이는 캡슐화된 State 객체들이 알아서 처리
  const result = trafficLight.request()
  output.value = result
}

const codeExample = `// 1. Context (문맥)
class TrafficLightContext {
  private currentState: TrafficLightState;

  constructor() {
    this.currentState = new RedState(); // 초기 상태
  }

  // State 객체가 Context의 상태를 변경할 수 있도록 함
  public setState(newState: TrafficLightState): void {
    this.currentState = newState;
  }

  // 클라이언트는 이 메서드만 호출
  public request(): void {
    // 실제 행동은 현재 State 객체에게 위임
    this.currentState.handle(this);
  }
}

// 2. State (상태 인터페이스)
interface TrafficLightState {
  // Context를 인자로 받아, 다음 상태로 전이시킬 수 있게 함
  handle(context: TrafficLightContext): void;
}

// 3. ConcreteState (구체적인 상태)
class RedState implements TrafficLightState {
  public handle(context: TrafficLightContext): void {
    console.log("🔴 정지 (STOP)");
    // 행동 수행 후, Context의 상태를 다음 상태(Green)로 변경
    context.setState(new GreenState());
  }
}

class GreenState implements TrafficLightState {
  public handle(context: TrafficLightContext): void {
    console.log("🟢 진행 (GO)");
    // 다음 상태(Yellow)로 변경
    context.setState(new YellowState());
  }
}

class YellowState implements TrafficLightState {
  public handle(context: TrafficLightContext): void {
    console.log("🟡 주의 (CAUTION)");
    // 다음 상태(Red)로 변경
    context.setState(new RedState());
  }
}

// --- 사용 ---
const light = new TrafficLightContext();
light.request(); // "🔴 정지 (STOP)" (상태가 Green으로 바뀜)
light.request(); // "🟢 진행 (GO)" (상태가 Yellow로 바뀜)
light.request(); // "🟡 주의 (CAUTION)" (상태가 Red로 바뀜)
`
</script>

<style scoped>
/* 이전과 동일한 스타일 재사용 */
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
  background: #c0392b; /* 상태 버튼 색상 변경 (Red) */
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
}
.test-btn:hover {
  background: #a93226;
}
.output {
  margin-top: 1.5rem;
  background: #2c3e50;
  color: #ecf0f1;
  padding: 1rem;
  border-radius: 4px;
  white-space: pre-wrap;
  font-size: 1.2rem; /* 상태 표시 크게 */
  text-align: center;
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
