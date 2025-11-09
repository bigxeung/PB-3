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

    <div class="comparison-section">
      <h3>📊 비교: 올바른 방법 vs 잘못된 방법</h3>
      <p>상태 패턴을 사용했을 때와 사용하지 않았을 때의 차이를 확인해보세요.</p>

      <div class="button-group">
        <button @click="showGoodExample" class="good-btn">✅ 올바른 방법 (상태 패턴)</button>
        <button @click="showBadExample" class="bad-btn">❌ 잘못된 방법 (if/else 분기)</button>
      </div>

      <div class="output good-output" v-if="goodOutput">
        <div class="output-header">✅ 올바른 방법: 상태 패턴 사용</div>
        <pre>{{ goodOutput }}</pre>
        <div class="explanation">
          💡 <strong>장점:</strong> 각 상태의 행동이 State 클래스로 분리되어 있어 새로운 상태 추가 시 기존 코드 수정이 불필요합니다. 상태 전이 로직이 명확합니다.
        </div>
      </div>

      <div class="output bad-output" v-if="badOutput">
        <div class="output-header">❌ 잘못된 방법: if/else 조건문 사용</div>
        <pre>{{ badOutput }}</pre>
        <div class="explanation">
          ⚠️ <strong>문제점:</strong> 모든 상태 로직이 하나의 클래스에 집중되어 복잡해지고, 새로운 상태 추가 시 기존 코드를 수정해야 합니다.
        </div>
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
const goodOutput = ref<string>('')
const badOutput = ref<string>('')
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

const showGoodExample = () => {
  badOutput.value = ''

  const results = [
    '--- 상태 패턴 사용 ---',
    '',
    'Context 클래스:',
    '  - 현재 상태만 보유',
    '  - request() 메서드 하나만 존재',
    '  - 상태별 로직 없음 (위임)',
    '',
    '각 State 클래스:',
    '  - RedState: 정지 행동 + Green으로 전이',
    '  - GreenState: 진행 행동 + Yellow로 전이',
    '  - YellowState: 주의 행동 + Red로 전이',
    '',
    '새로운 상태 추가 (BlinkingState):',
    '  - BlinkingState 클래스 추가',
    '  - Context 수정 불필요',
    '  - 다른 State 수정 불필요',
    '',
    '✅ 각 상태의 로직이 분리되어 유지보수 쉬움',
  ]

  goodOutput.value = results.join('\n')
}

const showBadExample = () => {
  goodOutput.value = ''

  const results = [
    '--- if/else 조건문 사용 (상태 패턴 미사용) ---',
    '',
    'class TrafficLight {',
    '  private currentState: string;',
    '',
    '  request() {',
    '    if (currentState === "red") {',
    '      console.log("🔴 정지");',
    '      currentState = "green";',
    '    } else if (currentState === "green") {',
    '      console.log("🟢 진행");',
    '      currentState = "yellow";',
    '    } else if (currentState === "yellow") {',
    '      console.log("🟡 주의");',
    '      currentState = "red";',
    '    }',
    '  }',
    '}',
    '',
    '새로운 상태 추가 (blinking):',
    '  - request() 메서드에 else if 추가',
    '  - 기존 코드 수정 필요',
    '  - 조건문이 길어지고 복잡해짐',
    '',
    '❌ 모든 로직이 한 곳에 집중되어 복잡함',
  ]

  badOutput.value = results.join('\n')
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
/* 상태 패턴 - 라벤더 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #f0f0ff 0%, #e6e6ff 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(224, 195, 252, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '💙';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #7c8aff;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(124, 138, 255, 0.1);
}

.description {
  color: #6b7bff;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #e0c3fc;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(224, 195, 252, 0.1);
}

.example-section,
.comparison-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(142, 197, 252, 0.2);
  box-shadow: 0 8px 25px rgba(142, 197, 252, 0.1);
}

.example-section h3,
.comparison-section h3 {
  margin-top: 0;
  color: #8e9aff;
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
  background: linear-gradient(135deg, #e0c3fc 0%, #8ec5fc 100%);
  color: white;
  box-shadow: 0 6px 20px rgba(142, 197, 252, 0.3);
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(142, 197, 252, 0.4);
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
  background: linear-gradient(135deg, #6b7bff 0%, #8e9aff 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 1.2rem;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(107, 123, 255, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
  text-align: center;
}

.good-output {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
  border: 3px solid rgba(67, 233, 123, 0.3);
  box-shadow: 0 8px 25px rgba(67, 233, 123, 0.3);
  text-align: left;
  font-size: 15px;
}

.bad-output {
  background: linear-gradient(135deg, #ff6b6b 0%, #ff8e53 100%);
  border: 3px solid rgba(255, 107, 107, 0.3);
  box-shadow: 0 8px 25px rgba(255, 107, 107, 0.3);
  text-align: left;
  font-size: 15px;
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
  border: 3px solid rgba(224, 195, 252, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #e0c3fc;
  margin-top: 0;
  border-bottom: 2px solid #e0c3fc;
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
