<template>
  <div class="pattern-container">
    <h2>5. 데코레이터 패턴 (Decorator Pattern)</h2>
    <p class="description">
      객체의 기존 코드를 변경하지 않으면서, 동적으로 새로운 책임(기능)을 객체에 덧붙이는 구조
      패턴입니다.<br />
      객체를 여러 겹의 데코레이터 객체로 감싸서 기능을 유연하게 확장합니다.
    </p>

    <div class="example-section">
      <h3>커피 주문 시스템</h3>
      <p>기본 커피(에스프레소)에 원하는 옵션을 추가해 보세요.</p>
      <div class="options">
        <label>
          <input type="checkbox" v-model="options.milk" />
          우유 추가 (+500원)
        </label>
        <label>
          <input type="checkbox" v-model="options.sugar" />
          설탕 추가 (+200원)
        </label>
        <label>
          <input type="checkbox" v-model="options.shot" />
          샷 추가 (+700원)
        </label>
      </div>
      <div class="button-group">
        <button @click="makeCoffee" class="test-btn">주문하기</button>
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
import { ref, reactive } from 'vue'

// --- 1. Component (컴포넌트 인터페이스) ---
// 원본 객체와 데코레이터가 공통으로 구현할 인터페이스입니다.
interface Coffee {
  getCost(): number
  getDescription(): string
}

// --- 2. ConcreteComponent (구체적인 컴포넌트) ---
// 장식될 기본 객체입니다. (예: 기본 커피)
class SimpleEspresso implements Coffee {
  public getCost(): number {
    return 3000
  }
  public getDescription(): string {
    return '에스프레소'
  }
}

// --- 3. Decorator (데코레이터 추상 클래스) ---
// Component 인터페이스를 구현하며,
// 내부에 Component 객체(wrapped)를 참조합니다.
abstract class CoffeeDecorator implements Coffee {
  protected wrappedCoffee: Coffee // 감쌀 객체(Component)

  constructor(coffee: Coffee) {
    this.wrappedCoffee = coffee
  }

  // 기본적으로는 감싸고 있는 객체에 위임
  public getCost(): number {
    return this.wrappedCoffee.getCost()
  }

  public getDescription(): string {
    return this.wrappedCoffee.getDescription()
  }
}

// --- 4. ConcreteDecorator (구체적인 데코레이터) ---
// 부모(Decorator)의 메서드를 오버라이드하여,
// 감싸고 있는 객체에 새로운 책임(기능)을 추가합니다.

// 우유 추가 데코레이터
class MilkDecorator extends CoffeeDecorator {
  constructor(coffee: Coffee) {
    super(coffee) // 부모 생성자 호출
  }

  // 부모 기능(기존 가격)에 새로운 기능(우유 가격)을 추가
  public getCost(): number {
    return super.getCost() + 500
  }

  // 부모 기능(기존 설명)에 새로운 기능(우유 설명)을 추가
  public getDescription(): string {
    return super.getDescription() + ' + 우유'
  }
}

// 설탕 추가 데코레이터
class SugarDecorator extends CoffeeDecorator {
  constructor(coffee: Coffee) {
    super(coffee)
  }

  public getCost(): number {
    return super.getCost() + 200
  }

  public getDescription(): string {
    return super.getDescription() + ' + 설탕'
  }
}

// 샷 추가 데코레이터
class ShotDecorator extends CoffeeDecorator {
  constructor(coffee: Coffee) {
    super(coffee)
  }

  public getCost(): number {
    return super.getCost() + 700
  }

  public getDescription(): string {
    return super.getDescription() + ' + 샷 추가'
  }
}

// --- Vue 로직 ---

const options = reactive({
  milk: false,
  sugar: false,
  shot: false,
})
const output = ref<string>('')

const makeCoffee = () => {
  // 1. 기본 객체(ConcreteComponent)로 시작
  let coffee: Coffee = new SimpleEspresso()

  // 2. 선택된 옵션에 따라 데코레이터로 동적으로 감싼다
  if (options.milk) {
    coffee = new MilkDecorator(coffee) // 우유 데코레이터로 감쌈
  }
  if (options.sugar) {
    coffee = new SugarDecorator(coffee) // 설탕 데코레이터로 감쌈
  }
  if (options.shot) {
    coffee = new ShotDecorator(coffee) // 샷 데코레이터로 감쌈
  }

  // 3. 최종적으로 감싸진 객체의 메서드 호출
  const cost = coffee.getCost()
  const description = coffee.getDescription()

  output.value = `주문 내역: ${description}\n총 가격: ${cost}원`
}

const codeExample = `// 1. Component (공통 인터페이스)
interface Coffee {
  getCost(): number;
  getDescription(): string;
}

// 2. ConcreteComponent (기본 객체)
class SimpleEspresso implements Coffee {
  public getCost(): number { return 3000; }
  public getDescription(): string { return '에스프레소'; }
}

// 3. Decorator (추상 데코레이터)
abstract class CoffeeDecorator implements Coffee {
  protected wrappedCoffee: Coffee;

  constructor(coffee: Coffee) {
    this.wrappedCoffee = coffee;
  }

  public getCost(): number {
    return this.wrappedCoffee.getCost(); // 원본 객체에 위임
  }
  public getDescription(): string {
    return this.wrappedCoffee.getDescription(); // 원본 객체에 위임
  }
}

// 4. ConcreteDecorator (기능 추가)
class MilkDecorator extends CoffeeDecorator {
  public getCost(): number {
    return super.getCost() + 500; // 원본 가격 + 우유 가격
  }
  public getDescription(): string {
    return super.getDescription() + ' + 우유'; // 원본 설명 + 우유
  }
}
// (SugarDecorator, ShotDecorator 등...)

// --- 사용 ---
// 기본 커피 생성
let coffee: Coffee = new SimpleEspresso();

// 옵션에 따라 동적으로 객체를 감싼다 (장식한다)
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

// 최종 결과물은 모든 기능이 합쳐져서 나온다.
console.log(coffee.getDescription()); // "에스프레소 + 우유 + 설탕"
console.log(coffee.getCost()); // 3000 + 500 + 200 = 3700
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
/* 옵션 체크박스 스타일 */
.options {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin-top: 1rem;
  margin-bottom: 1.5rem;
}
.options label {
  font-size: 16px;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.button-group {
  display: flex;
  gap: 1rem;
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
