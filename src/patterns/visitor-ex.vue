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
  flex-wrap: wrap;
}
.test-btn {
  background: #9b59b6; /* 비지터 버튼 색상 변경 */
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
}
.test-btn:hover {
  background: #8e44ad;
}
.test-btn:nth-of-type(2) {
  background: #e67e22;
}
.test-btn:nth-of-type(2):hover {
  background: #d35400;
}
.output {
  margin-top: 1.5rem;
  background: #2c3e50;
  color: #ecf0f1;
  padding: 1rem;
  border-radius: 4px;
  white-space: pre-wrap;
  max-height: 300px;
  overflow-y: auto;
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
