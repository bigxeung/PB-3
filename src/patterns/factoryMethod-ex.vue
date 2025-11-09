<template>
  <div class="pattern-container">
    <h2>2. 팩토리 메서드 패턴 (Factory Method)</h2>
    <p class="description">
      객체를 생성하는 인터페이스를 정의하고, 실제 인스턴스 생성은 서브클래스에게 맡기는
      패턴입니다.<br />
      어떤 객체를 생성할지가 서브클래스에 의해 결정됩니다.
    </p>

    <div class="example-section">
      <h3>운송 시뮬레이터</h3>
      <div class="button-group">
        <button @click="runLogistics('road')" class="test-btn">육상 운송 (트럭) 배송 시작</button>
        <button @click="runLogistics('sea')" class="test-btn">해상 운송 (배) 배송 시작</button>
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

// --- 1. Product (제품) ---
// 생성될 객체의 공통 인터페이스입니다.
interface Transport {
  deliver(): string
}

// --- 2. ConcreteProduct (구체적인 제품) ---
// Transport 인터페이스를 구현한 실제 객체입니다.
class Truck implements Transport {
  public deliver(): string {
    return '트럭이 육로로 배송을 시작합니다... 🚚'
  }
}

class Ship implements Transport {
  public deliver(): string {
    return '배가 해상으로 배송을 시작합니다... 🚢'
  }
}

// --- 3. Creator (생성자) ---
// 팩토리 메서드(createTransport)를 선언하는 추상 클래스입니다.
// 이 클래스는 자신이 생성할 객체의 클래스를 알지 못합니다.
abstract class Logistics {
  // 팩토리 메서드 (서브클래스가 구현해야 함)
  public abstract createTransport(): Transport

  // 팩토리 메서드를 사용하여 실제 작업을 수행
  public planDelivery(): string {
    // 팩토리 메서드를 호출하여 Product 객체를 생성
    const transport = this.createTransport()
    // 생성된 Product 객체의 메서드를 사용
    return `[계획 실행] ${transport.deliver()}`
  }
}

// --- 4. ConcreteCreator (구체적인 생성자) ---
// 팩토리 메서드를 실제로 구현하여 구체적인 제품을 생성합니다.

// 육상 운송 Creator (트럭을 생성)
class RoadLogistics extends Logistics {
  public createTransport(): Transport {
    return new Truck()
  }
}

// 해상 운송 Creator (배를 생성)
class SeaLogistics extends Logistics {
  public createTransport(): Transport {
    return new Ship()
  }
}

// --- Vue 로직 ---

const output = ref<string>('')
let logistics: Logistics // Creator 타입으로 변수 선언

const runLogistics = (type: 'road' | 'sea') => {
  if (type === 'road') {
    // 육상 운송 Creator를 사용
    logistics = new RoadLogistics()
  } else {
    // 해상 운송 Creator를 사용
    logistics = new SeaLogistics()
  }

  // 어떤 Creator가 선택되었든, 클라이언트는 동일한 planDelivery() 메서드를 호출
  output.value = logistics.planDelivery()
}

const codeExample = `// 1. Product (제품 인터페이스)
interface Transport {
  deliver(): string;
}

// 2. ConcreteProduct (구체적인 제품)
class Truck implements Transport {
  public deliver(): string { /* ... */ }
}
class Ship implements Transport {
  public deliver(): string { /* ... */ }
}

// 3. Creator (생성자 추상 클래스)
abstract class Logistics {
  // 팩토리 메서드 (서브클래스가 구현)
  public abstract createTransport(): Transport;

  // 팩토리 메서드를 사용하는 로직
  public planDelivery(): string {
    const transport = this.createTransport();
    return transport.deliver();
  }
}

// 4. ConcreteCreator (구체적인 생성자)
class RoadLogistics extends Logistics {
  public createTransport(): Transport {
    return new Truck(); // 트럭 생성
  }
}
class SeaLogistics extends Logistics {
  public createTransport(): Transport {
    return new Ship(); // 배 생성
  }
}

// --- 사용 ---
// 클라이언트는 Creator의 타입(Road/Sea)만 교체하면
// planDelivery() 수정 없이 다른 제품을 생성하고 사용할 수 있습니다.
let logistics: Logistics;
logistics = new RoadLogistics();
console.log(logistics.planDelivery()); // 트럭 배송

logistics = new SeaLogistics();
console.log(logistics.planDelivery()); // 배 배송
`
</script>

<style scoped>
/* 사용자님이 제공한 스타일을 그대로 적용합니다 */
.pattern-container {
  padding: 2rem;
  max-width: 900px;
  margin: 20px auto; /* 위아래 간격 추가 */
  border: 1px solid #e0e0e0; /* 구분을 위한 테두리 */
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
  background-color: #f8f9fa; /* 설명 배경 */
  padding: 16px;
  border-radius: 8px;
}

.example-section {
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  margin-bottom: 2rem;
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
/* 두 번째 버튼 스타일 */
.test-btn:nth-of-type(2) {
  background: #3498db;
}
.test-btn:nth-of-type(2):hover {
  background: #2980b9;
}

.output {
  margin-top: 1.5rem; /* 버튼과의 간격 */
  background: #2c3e50;
  color: #ecf0f1;
  padding: 1rem;
  border-radius: 4px;
  white-space: pre-wrap; /* 줄바꿈 적용 */
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
