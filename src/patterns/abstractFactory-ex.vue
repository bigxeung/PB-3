<template>
  <div class="pattern-container">
    <h2>3. 추상 팩토리 패턴 (Abstract Factory)</h2>
    <p class="description">
      서로 관련이 있거나 의존적인 객체들의 '군(Family)'을 생성하기 위한 인터페이스를 제공하는
      패턴입니다.<br />
      팩토리의 구체적인 클래스를 교체하면, 생성되는 객체들의 '테마'가 한 번에 변경됩니다.
    </p>

    <div class="example-section">
      <h3>GUI 팩토리 시뮬레이터</h3>
      <p>애플리케이션의 현재 테마(OS)를 선택하세요:</p>
      <div class="button-group">
        <button @click="createUI('Windows')" class="test-btn">Windows 테마</button>
        <button @click="createUI('Mac')" class="test-btn">macOS 테마</button>
      </div>
      <div class="output" v-if="output">
        <pre>{{ output }}</pre>
      </div>
    </div>

    <div class="comparison-section">
      <h3>📊 비교: 올바른 방법 vs 잘못된 방법</h3>
      <p>추상 팩토리 패턴을 사용했을 때와 사용하지 않았을 때의 차이를 확인해보세요.</p>

      <div class="button-group">
        <button @click="showGoodExample" class="good-btn">✅ 올바른 방법 (추상 팩토리 사용)</button>
        <button @click="showBadExample" class="bad-btn">❌ 잘못된 방법 (직접 조합)</button>
      </div>

      <div class="output good-output" v-if="goodOutput">
        <div class="output-header">✅ 올바른 방법: 추상 팩토리 사용</div>
        <pre>{{ goodOutput }}</pre>
        <div class="explanation">
          💡 <strong>장점:</strong> 팩토리가 관련된 제품군의 일관성을 보장합니다. 테마를 변경해도 모든 UI 요소가 자동으로 동일한 스타일로 유지됩니다.
        </div>
      </div>

      <div class="output bad-output" v-if="badOutput">
        <div class="output-header">❌ 잘못된 방법: 클라이언트가 직접 제품 조합</div>
        <pre>{{ badOutput }}</pre>
        <div class="explanation">
          ⚠️ <strong>문제점:</strong> 클라이언트가 각 제품을 직접 선택하면 일관성 없는 조합(Windows 버튼 + Mac 체크박스)이 발생할 수 있습니다. UI 일관성이 깨집니다.
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
import { ref } from 'vue'

// --- 1. AbstractProduct (추상 제품) ---
// 생성될 객체들의 공통 인터페이스입니다.

interface Button {
  paint(): string
}

interface Checkbox {
  paint(): string
}

// --- 2. ConcreteProduct (구체적인 제품) ---
// OS 테마별로 실제 제품을 구현합니다.

// Windows 제품군
class WinButton implements Button {
  public paint(): string {
    return 'Windows 스타일 버튼 렌더링 [ 네모 버튼 ]'
  }
}
class WinCheckbox implements Checkbox {
  public paint(): string {
    return 'Windows 스타일 체크박스 렌더링 [ V 체크박스 ]'
  }
}

// Mac 제품군
class MacButton implements Button {
  public paint(): string {
    return 'macOS 스타일 버튼 렌더링 ( 둥근 버튼 )'
  }
}
class MacCheckbox implements Checkbox {
  public paint(): string {
    return 'macOS 스타일 체크박스 렌더링 ( O 체크박스 )'
  }
}

// --- 3. AbstractFactory (추상 팩토리) ---
// 관련 제품군을 생성하는 메서드들의 인터페이스입니다.
interface GUIFactory {
  createButton(): Button
  createCheckbox(): Checkbox
}

// --- 4. ConcreteFactory (구체적인 팩토리) ---
// 추상 팩토리를 구현하여, 특정 테마의 제품군을 생성합니다.

// Windows 팩토리 (Windows 제품군만 생성)
class WinFactory implements GUIFactory {
  public createButton(): Button {
    return new WinButton()
  }
  public createCheckbox(): Checkbox {
    return new WinCheckbox()
  }
}

// Mac 팩토리 (Mac 제품군만 생성)
class MacFactory implements GUIFactory {
  public createButton(): Button {
    return new MacButton()
  }
  public createCheckbox(): Checkbox {
    return new MacCheckbox()
  }
}

// --- 5. 클라이언트 ---
// 추상 팩토리와 추상 제품의 인터페이스만 알고 사용합니다.
class Application {
  private factory: GUIFactory
  private button!: Button
  private checkbox!: Checkbox

  constructor(factory: GUIFactory) {
    this.factory = factory
  }

  // 팩토리를 사용해 UI 요소를 생성
  public createUI(): void {
    this.button = this.factory.createButton()
    this.checkbox = this.factory.createCheckbox()
  }

  // 생성된 UI 요소를 사용
  public paintUI(): string {
    return [this.button.paint(), this.checkbox.paint()].join('\n')
  }
}

// --- Bad Example: 클라이언트가 직접 제품 조합 ---
class BadClient {
  public createMixedUI(): string {
    // 문제: 서로 다른 테마의 제품을 섞어서 사용 (일관성 없음)
    const winButton = new WinButton() // Windows 버튼
    const macCheckbox = new MacCheckbox() // Mac 체크박스

    return [
      winButton.paint(),
      macCheckbox.paint(),
      '',
      '⚠️ Windows 버튼과 Mac 체크박스가 섞여있습니다!',
      '⚠️ UI 일관성이 깨졌습니다!',
    ].join('\n')
  }
}

// --- Vue 로직 ---

const output = ref<string>('')
const goodOutput = ref<string>('')
const badOutput = ref<string>('')
let factory: GUIFactory
let app: Application

const createUI = (os: 'Windows' | 'Mac') => {
  // 클라이언트는 팩토리를 주입받아 Application을 생성
  if (os === 'Windows') {
    factory = new WinFactory()
  } else {
    factory = new MacFactory()
  }

  // 주입된 팩토리에 따라 Application의 행동이 결정됨
  app = new Application(factory)

  // UI 생성 및 렌더링
  app.createUI()
  output.value = `[ ${os} 테마 적용됨 ]\n` + app.paintUI()
}

const showGoodExample = () => {
  badOutput.value = '' // 다른 출력 숨기기

  // 추상 팩토리 사용: 팩토리가 일관된 제품군 보장
  const winFactory: GUIFactory = new WinFactory()
  const macFactory: GUIFactory = new MacFactory()

  const winApp = new Application(winFactory)
  winApp.createUI()

  const macApp = new Application(macFactory)
  macApp.createUI()

  const results = [
    '--- 추상 팩토리 패턴 사용 ---',
    '',
    '1. Windows 테마:',
    winApp.paintUI(),
    '',
    '2. macOS 테마:',
    macApp.paintUI(),
    '',
    '✅ 각 팩토리가 일관된 제품군을 생성합니다.',
    '✅ Windows 팩토리는 Windows 제품만 생성',
    '✅ Mac 팩토리는 Mac 제품만 생성',
    '✅ 테마 일관성이 자동으로 보장됩니다!',
  ]

  goodOutput.value = results.join('\n')
}

const showBadExample = () => {
  goodOutput.value = '' // 다른 출력 숨기기

  const badClient = new BadClient()

  const results = [
    '--- 직접 조합 방식 (추상 팩토리 미사용) ---',
    '',
    badClient.createMixedUI(),
    '',
    '❌ 클라이언트가 각 제품을 직접 선택하면:',
    '   - Windows 버튼 + Mac 체크박스 같은 일관성 없는 조합 발생',
    '   - UI 테마가 뒤죽박죽 섞임',
    '   - 사용자 경험 저하',
  ]

  badOutput.value = results.join('\n')
}

const codeExample = `// 1. AbstractProduct (제품 인터페이스)
interface Button { paint(): string; }
interface Checkbox { paint(): string; }

// 2. ConcreteProduct (구체적인 제품)
class WinButton implements Button { /* ... */ }
class WinCheckbox implements Checkbox { /* ... */ }
class MacButton implements Button { /* ... */ }
class MacCheckbox implements Checkbox { /* ... */ }

// 3. AbstractFactory (추상 팩토리 인터페이스)
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// 4. ConcreteFactory (구체적인 팩토리)
// WinFactory는 Win 제품군만 생성
class WinFactory implements GUIFactory {
  public createButton(): Button { return new WinButton(); }
  public createCheckbox(): Checkbox { return new WinCheckbox(); }
}
// MacFactory는 Mac 제품군만 생성
class MacFactory implements GUIFactory {
  public createButton(): Button { return new MacButton(); }
  public createCheckbox(): Checkbox { return new MacCheckbox(); }
}

// --- 사용 ---
// 클라이언트는 어떤 팩토리를 주입받느냐에 따라
// 생성되는 제품군(테마)이 통째로 바뀝니다.
let factory: GUIFactory;

factory = new WinFactory();
const winApp = new Application(factory);
winApp.createUI();
winApp.paintUI(); // Windows 버튼/체크박스 렌더링

factory = new MacFactory();
const macApp = new Application(factory);
macApp.createUI();
macApp.paintUI(); // macOS 버튼/체크박스 렌더링
`
</script>

<style scoped>
/* 추상 팩토리 패턴 - 민트/그린 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #d9fff5 0%, #c7ffe8 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(67, 233, 123, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🌿';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #00b894;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(0, 184, 148, 0.1);
}

.description {
  color: #00a383;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #43e97b;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(67, 233, 123, 0.1);
}

.example-section,
.comparison-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(67, 233, 123, 0.2);
  box-shadow: 0 8px 25px rgba(67, 233, 123, 0.1);
}

.example-section h3,
.comparison-section h3 {
  margin-top: 0;
  color: #1dd1a1;
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
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
  color: white;
  box-shadow: 0 6px 20px rgba(67, 233, 123, 0.3);
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(67, 233, 123, 0.4);
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
  background: linear-gradient(135deg, #10ac84 0%, #1dd1a1 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(16, 172, 132, 0.3);
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
  border: 3px solid rgba(67, 233, 123, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #43e97b;
  margin-top: 0;
  border-bottom: 2px solid #43e97b;
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
