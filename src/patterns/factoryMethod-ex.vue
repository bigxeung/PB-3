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

    <div class="comparison-section">
      <h3>📊 비교: 올바른 방법 vs 잘못된 방법</h3>
      <p>팩토리 메서드 패턴을 사용했을 때와 사용하지 않았을 때의 차이를 확인해보세요.</p>

      <div class="button-group">
        <button @click="showGoodExample" class="good-btn">✅ 올바른 방법 (팩토리 메서드 사용)</button>
        <button @click="showBadExample" class="bad-btn">❌ 잘못된 방법 (직접 생성)</button>
      </div>

      <div class="output good-output" v-if="goodOutput">
        <div class="output-header">✅ 올바른 방법: 팩토리 메서드 사용</div>
        <pre>{{ goodOutput }}</pre>
        <div class="explanation">
          💡 <strong>장점:</strong> 클라이언트 코드는 추상 인터페이스만 사용하므로 느슨한 결합이 유지됩니다. 새로운 운송 수단(비행기 등)을 추가해도 클라이언트 코드 수정이 불필요합니다.
        </div>
      </div>

      <div class="output bad-output" v-if="badOutput">
        <div class="output-header">❌ 잘못된 방법: 클라이언트가 직접 구체 클래스 생성</div>
        <pre>{{ badOutput }}</pre>
        <div class="explanation">
          ⚠️ <strong>문제점:</strong> 클라이언트 코드가 구체 클래스(Truck, Ship)에 직접 의존하여 강한 결합이 발생합니다. 새로운 운송 수단을 추가하면 클라이언트 코드도 수정해야 합니다.
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

// --- Bad Example: 클라이언트가 직접 구체 클래스 생성 ---
class BadClient {
  public deliver(type: 'road' | 'sea'): string {
    // 클라이언트 코드가 구체 클래스에 직접 의존 (강한 결합)
    if (type === 'road') {
      const truck = new Truck() // 직접 생성
      return `[클라이언트] ${truck.deliver()}`
    } else {
      const ship = new Ship() // 직접 생성
      return `[클라이언트] ${ship.deliver()}`
    }
    // 문제점: 새로운 운송 수단 추가 시 이 클라이언트 코드를 수정해야 함
  }
}

// --- Vue 로직 ---

const output = ref<string>('')
const goodOutput = ref<string>('')
const badOutput = ref<string>('')
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

const showGoodExample = () => {
  badOutput.value = '' // 다른 출력 숨기기

  // 팩토리 메서드 패턴 사용: 클라이언트는 추상 타입만 사용
  const roadLogistics: Logistics = new RoadLogistics()
  const seaLogistics: Logistics = new SeaLogistics()

  const results = [
    '--- 팩토리 메서드 패턴 사용 ---',
    '',
    '1. 육상 운송:',
    roadLogistics.planDelivery(),
    '',
    '2. 해상 운송:',
    seaLogistics.planDelivery(),
    '',
    '✅ 클라이언트는 Logistics 인터페이스만 알면 됩니다.',
    '✅ 새로운 운송 수단(비행기) 추가 시:',
    '   - AirLogistics 클래스만 추가',
    '   - 클라이언트 코드는 수정 불필요',
  ]

  goodOutput.value = results.join('\n')
}

const showBadExample = () => {
  goodOutput.value = '' // 다른 출력 숨기기

  const badClient = new BadClient()

  const results = [
    '--- 직접 생성 방식 (팩토리 메서드 미사용) ---',
    '',
    '1. 육상 운송:',
    badClient.deliver('road'),
    '',
    '2. 해상 운송:',
    badClient.deliver('sea'),
    '',
    '❌ 클라이언트가 Truck, Ship 구체 클래스를 직접 알아야 합니다.',
    '❌ 새로운 운송 수단(비행기) 추가 시:',
    '   - Plane 클래스 추가',
    '   - BadClient의 deliver() 메서드에 else if 분기 추가 필요',
    '   - 클라이언트 코드 수정 필요 (강한 결합)',
  ]

  badOutput.value = results.join('\n')
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
/* 팩토리 메서드 패턴 - 블루 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #e0f7ff 0%, #d4f1ff 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(79, 172, 254, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🚢';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #0099ff;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(0, 153, 255, 0.1);
}

.description {
  color: #0077cc;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #4facfe;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(79, 172, 254, 0.1);
}

.example-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(79, 172, 254, 0.2);
  box-shadow: 0 8px 25px rgba(79, 172, 254, 0.1);
}

.example-section h3 {
  margin-top: 0;
  color: #00b4d8;
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
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 6px 20px rgba(79, 172, 254, 0.3);
  transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  margin-top: 0.5rem;
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(79, 172, 254, 0.4);
}
.test-btn:active {
  transform: translateY(0) scale(0.98);
}

.output {
  margin-top: 1.5rem;
  background: linear-gradient(135deg, #0077b6 0%, #0096c7 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(0, 119, 182, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
}

.code-section {
  background: linear-gradient(135deg, #1e1e2e 0%, #2a2a3e 100%);
  padding: 2rem;
  border-radius: 25px;
  overflow-x: auto;
  border: 3px solid rgba(79, 172, 254, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #4facfe;
  margin-top: 0;
  border-bottom: 2px solid #4facfe;
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
