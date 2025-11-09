<template>
  <div class="pattern-container">
    <h2>8. 비지터 패턴 (Visitor Pattern)</h2>
    <p class="description">
      객체 구조(Element)의 코드를 변경하지 않으면서, 해당 객체 구조에 새로운 연산(Visitor)을<br />
      추가할 수 있게 해주는 행위 패턴입니다. (데이터와 로직의 분리)
    </p>

    <div class="example-section">
      <h3>도형 계산기</h3>
      <p>
        도형(Circle, Square) 클래스의 코드를 수정하지 않고, '면적 계산'과 '둘레 계산' 기능을
        방문자(Visitor)로 추가합니다.
      </p>
      <div class="button-group">
        <button @click="runVisitor('area')" class="test-btn">모든 도형 면적 계산</button>
        <button @click="runVisitor('perimeter')" class="test-btn">모든 도형 둘레 계산</button>
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

// --- 1. Visitor (방문자 인터페이스) ---
// 각 Element를 방문(visit)하는 메서드를 정의합니다.
// (TypeScript의 메서드 오버로딩 한계로, 구체적인 visit 메서드를 정의)
interface ShapeVisitor {
  visitCircle(circle: Circle): string
  visitSquare(square: Square): string
}

// --- 2. Element (요소 인터페이스) ---
// Visitor를 받아들이는(accept) 메서드를 정의합니다.
interface Shape {
  accept(visitor: ShapeVisitor): string
}

// --- 3. ConcreteElement (구체적인 요소) ---
// Element 인터페이스를 구현합니다.
// (Circle과 Square는 자신의 코드를 변경할 필요가 없음)

class Circle implements Shape {
  public radius: number

  constructor(radius: number) {
    this.radius = radius
  }

  // 'accept' 메서드는 Visitor에게 자기 자신(this)을 넘김
  public accept(visitor: ShapeVisitor): string {
    return visitor.visitCircle(this)
  }
}

class Square implements Shape {
  public side: number

  constructor(side: number) {
    this.side = side
  }

  // 'accept' 메서드는 Visitor에게 자기 자신(this)을 넘김
  public accept(visitor: ShapeVisitor): string {
    return visitor.visitSquare(this)
  }
}

// --- 4. ConcreteVisitor (구체적인 방문자) ---
// Visitor 인터페이스를 구현하여, Element에 대한 새로운 로직을 추가합니다.

// '면적 계산' 방문자
class AreaCalculator implements ShapeVisitor {
  public visitCircle(circle: Circle): string {
    const area = Math.PI * circle.radius * circle.radius
    return `[면적] 원 (반지름 ${circle.radius}): ${area.toFixed(2)}`
  }

  public visitSquare(square: Square): string {
    const area = square.side * square.side
    return `[면적] 사각형 (변 ${square.side}): ${area.toFixed(2)}`
  }
}

// '둘레 계산' 방문자
class PerimeterCalculator implements ShapeVisitor {
  public visitCircle(circle: Circle): string {
    const perimeter = 2 * Math.PI * circle.radius
    return `[둘레] 원 (반지름 ${circle.radius}): ${perimeter.toFixed(2)}`
  }

  public visitSquare(square: Square): string {
    const perimeter = 4 * square.side
    return `[둘레] 사각형 (변 ${square.side}): ${perimeter.toFixed(2)}`
  }
}

// --- Vue 로직 ---

const output = ref<string>('')

// 1. Element(도형) 객체 구조 생성
const shapes: Shape[] = [
  new Circle(10), // 반지름 10인 원
  new Square(8), // 한 변이 8인 사각형
  new Circle(5), // 반지름 5인 원
]

const runVisitor = (type: 'area' | 'perimeter') => {
  let visitor: ShapeVisitor
  const results: string[] = []

  // 2. 실행할 로직(Visitor) 선택
  if (type === 'area') {
    visitor = new AreaCalculator()
    results.push('--- 면적 계산기(Visitor) 실행 ---')
  } else {
    visitor = new PerimeterCalculator()
    results.push('--- 둘레 계산기(Visitor) 실행 ---')
  }

  // 3. 객체 구조(shapes)를 순회하며 Visitor를 주입
  // (Shape 클래스 코드 수정 없이, 로직(Visitor)만 교체하여 실행)
  for (const shape of shapes) {
    results.push(shape.accept(visitor))
  }

  output.value = results.join('\n')
}

const codeExample = `// 1. Visitor (방문자 인터페이스)
interface ShapeVisitor {
  visitCircle(circle: Circle): void;
  visitSquare(square: Square): void;
}

// 2. Element (요소 인터페이스)
interface Shape {
  accept(visitor: ShapeVisitor): void;
}

// 3. ConcreteElement (구체적인 요소)
class Circle implements Shape {
  public radius: number;
  // ...
  public accept(visitor: ShapeVisitor): void {
    // Visitor에게 자신(Circle)을 처리해달라고 호출 (더블 디스패치)
    visitor.visitCircle(this);
  }
}
class Square implements Shape {
  public side: number;
  // ...
  public accept(visitor: ShapeVisitor): void {
    visitor.visitSquare(this);
  }
}

// 4. ConcreteVisitor (구체적인 방문자 - 로직)
// (Element 클래스를 수정하지 않고, 새로운 로직(면적)을 추가)
class AreaCalculator implements ShapeVisitor {
  public visitCircle(circle: Circle): void {
    const area = Math.PI * circle.radius * circle.radius;
    console.log(\`원 면적: \${area}\`);
  }
  public visitSquare(square: Square): void {
    const area = square.side * square.side;
    console.log(\`사각형 면적: \${area}\`);
  }
}
// (PerimeterCalculator 등 다른 로직도 Visitor로 추가 가능)

// --- 사용 ---
const shapes: Shape[] = [new Circle(10), new Square(5)];
const areaVisitor = new AreaCalculator();

// shapes 구조를 순회하며 areaVisitor를 주입
for (const shape of shapes) {
  shape.accept(areaVisitor);
}
`
</script>

<style scoped>
/* 비지터 패턴 - 피치 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #fff9f0 0%, #ffeddb 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(255, 236, 210, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🍑';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #ff9966;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(255, 153, 102, 0.1);
}

.description {
  color: #ff8855;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #ffecd2;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(255, 236, 210, 0.1);
}

.example-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(252, 182, 159, 0.2);
  box-shadow: 0 8px 25px rgba(252, 182, 159, 0.1);
}

.example-section h3 {
  margin-top: 0;
  color: #ffaa77;
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
  background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 6px 20px rgba(252, 182, 159, 0.3);
  transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  margin-top: 0.5rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(252, 182, 159, 0.4);
}
.test-btn:active {
  transform: translateY(0) scale(0.98);
}

.output {
  margin-top: 1.5rem;
  background: linear-gradient(135deg, #ff9966 0%, #ff8c5a 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(255, 153, 102, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
  max-height: 300px;
  overflow-y: auto;
}

.code-section {
  background: linear-gradient(135deg, #1e1e2e 0%, #2a2a3e 100%);
  padding: 2rem;
  border-radius: 25px;
  overflow-x: auto;
  border: 3px solid rgba(252, 182, 159, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #ffecd2;
  margin-top: 0;
  border-bottom: 2px solid #ffecd2;
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
