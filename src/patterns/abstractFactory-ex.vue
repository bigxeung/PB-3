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

// --- Vue 로직 ---

const output = ref<string>('')
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
  background: #42b983;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
}
.test-btn:hover {
  background: #36a473;
}
.test-btn:nth-of-type(2) {
  background: #3498db;
}
.test-btn:nth-of-type(2):hover {
  background: #2980b9;
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
