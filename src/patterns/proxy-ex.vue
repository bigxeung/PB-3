<template>
  <div class="pattern-container">
    <h2>6. 프록시 패턴 (Proxy Pattern)</h2>
    <p class="description">
      다른 객체에 대한 접근을 제어하기 위해 대리자(placeholder)를 제공하는 구조 패턴입니다.<br />
      무거운 객체의 로딩을 지연(Lazy Initialization)시키거나, 접근 권한을 제어할 수 있습니다.
    </p>

    <div class="example-section">
      <h3>고해상도 이미지 로더 (지연 초기화 프록시)</h3>
      <p>
        프록시 객체는 생성 시점이 아닌, <code>display()</code>가 처음 호출될 때 실제 이미지
        로더(무거운 객체)를 생성합니다.
      </p>
      <div class="button-group">
        <button @click="loadImage(1)" class="test-btn">이미지 1 표시</button>
        <button @click="loadImage(2)" class="test-btn">이미지 2 표시</button>
        <button @click="loadImage(1)" class="test-btn alt">이미지 1 다시 표시</button>
      </div>
      <div class="output" v-if="output.length">
        <pre>{{ output.join('\n') }}</pre>
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

// --- 1. Subject (주제 인터페이스) ---
// RealSubject와 Proxy가 공통으로 구현할 인터페이스입니다.
interface Image {
  display(): string
}

// --- 2. RealSubject (실제 주제) ---
// 리소스를 많이 소모하는 '무거운' 객체입니다.
// 이 예제에서는 생성자(constructor)가 오래 걸린다고 가정합니다.
class RealImage implements Image {
  private filename: string
  private loadLog: string

  constructor(filename: string) {
    this.filename = filename
    // 시뮬레이션: 디스크에서 이미지를 로드하는 무거운 작업
    this.loadLog = `[RealImage] ${this.filename} 로딩 완료... (무거운 작업)`
    console.log(this.loadLog)
  }

  public display(): string {
    return `[RealImage] ${this.filename} 화면에 표시 🖼️`
  }

  public getLoadLog(): string {
    return this.loadLog
  }
}

// --- 3. Proxy (프록시) ---
// Subject 인터페이스를 구현하며, RealSubject 객체를 내부에 참조합니다.
// 클라이언트는 이 프록시 객체를 RealSubject처럼 사용합니다.
class ProxyImage implements Image {
  private realImage: RealImage | null = null // RealSubject 참조 (처음엔 null)
  private filename: string

  constructor(filename: string) {
    this.filename = filename
    console.log(`[ProxyImage] 프록시 생성됨 (아직 로딩 안 함): ${filename}`)
  }

  // RealSubject의 생성을 display()가 호출될 때까지 지연 (Lazy Initialization)
  public display(): string {
    const logs: string[] = []

    logs.push(`[ProxyImage] display() 호출됨 (${this.filename})`)

    // 1. RealImage가 아직 생성되지 않았다면, 이 시점에 생성
    if (this.realImage === null) {
      logs.push(`[ProxyImage] RealImage가 없으므로 새로 생성...`)
      this.realImage = new RealImage(this.filename)
      logs.push(this.realImage.getLoadLog()) // 로딩 로그 추가
    } else {
      logs.push(`[ProxyImage] 이미 로드된 RealImage 재사용.`)
    }

    // 2. 실제 객체에 작업 위임
    logs.push(this.realImage.display())
    return logs.join('\n')
  }
}

// --- Vue 로직 ---

const output = ref<string[]>([])

// Vue 컴포넌트가 마운트될 때 프록시 객체들을 미리 생성
// ※주의: 이 시점에는 '무거운' RealImage는 생성되지 않습니다. (콘솔 로그 확인)
let image1: Image
let image2: Image

onMounted(() => {
  image1 = new ProxyImage('Vacation_Photo_001.jpg')
  image2 = new ProxyImage('Project_Diagram_002.png')
  output.value.push('--- 프록시 객체 2개 생성 완료 (아직 로딩 전) ---')
})

const loadImage = (imageNumber: 1 | 2) => {
  output.value.push(`\n--- [버튼 클릭: 이미지 ${imageNumber}] ---`)
  if (imageNumber === 1) {
    // image1.display() 호출
    // 첫 호출 시: RealImage 생성 (무거운 로딩 발생)
    // 두 번째 호출 시: 기존 RealImage 재사용 (로딩 없음)
    output.value.push(image1.display())
  } else {
    // image2.display() 호출
    output.value.push(image2.display())
  }
}

const codeExample = `// 1. Subject (공통 인터페이스)
interface Image {
  display(): string;
}

// 2. RealSubject (무거운 객체)
class RealImage implements Image {
  constructor(filename: string) {
    // (시뮬레이션: 무거운 로딩 작업)
    console.log(\`\${filename} 로딩 중...\`);
  }
  public display(): string { /* ... */ }
}

// 3. Proxy (대리자)
class ProxyImage implements Image {
  private realImage: RealImage | null = null; // 실제 객체 참조
  private filename: string;

  constructor(filename: string) {
    this.filename = filename;
  }

  // 실제 작업(display)이 요청될 때
  public display(): string {
    // RealImage가 아직 없으면(null) 그 때 생성 (지연 초기화)
    if (this.realImage === null) {
      this.realImage = new RealImage(this.filename);
    }

    // 실제 객체에 작업 위임
    return this.realImage.display();
  }
}

// --- 사용 ---
// 프록시 생성은 매우 빠름 (RealImage 로딩 안 함)
const image = new ProxyImage("photo.jpg");

// 이 시점(display 호출 시)에 RealImage가 생성되고 로딩됨
console.log(image.display());

// 이 시점에는 이미 생성된 RealImage를 재사용 (로딩 없음)
console.log(image.display());
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
  flex-wrap: wrap; /* 버튼이 많아지면 줄바꿈 */
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
.test-btn.alt {
  /* 세 번째 버튼 */
  background: #e67e22;
}
.test-btn.alt:hover {
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
  overflow-y: auto; /* 출력이 길어지면 스크롤 */
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
