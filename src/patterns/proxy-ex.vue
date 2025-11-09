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

    <div class="comparison-section">
      <h3>📊 비교: 올바른 방법 vs 잘못된 방법</h3>
      <p>프록시 패턴을 사용했을 때와 사용하지 않았을 때의 차이를 확인해보세요.</p>

      <div class="button-group">
        <button @click="showGoodExample" class="good-btn">✅ 올바른 방법 (프록시 사용)</button>
        <button @click="showBadExample" class="bad-btn">❌ 잘못된 방법 (직접 생성)</button>
      </div>

      <div class="output good-output" v-if="goodOutput">
        <div class="output-header">✅ 올바른 방법: 프록시로 지연 초기화</div>
        <pre>{{ goodOutput }}</pre>
        <div class="explanation">
          💡 <strong>장점:</strong> 프록시를 사용하면 객체 생성을 실제로 필요한 시점까지 지연시켜 초기 로딩 시간을 줄이고 메모리를 절약할 수 있습니다.
        </div>
      </div>

      <div class="output bad-output" v-if="badOutput">
        <div class="output-header">❌ 잘못된 방법: 즉시 모든 객체 생성</div>
        <pre>{{ badOutput }}</pre>
        <div class="explanation">
          ⚠️ <strong>문제점:</strong> 사용하지 않을 수도 있는 무거운 객체를 미리 모두 생성하면 초기 로딩 시간이 길어지고 메모리가 낭비됩니다.
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
const goodOutput = ref<string>('')
const badOutput = ref<string>('')

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

const showGoodExample = () => {
  badOutput.value = ''

  const results = [
    '--- 프록시 패턴 사용 ---',
    '',
    '1. 초기화 시점:',
    '   프록시 3개 생성 (빠름, 메모리 적게 사용)',
    '   RealImage는 아직 생성되지 않음',
    '',
    '2. 이미지 1 표시 요청:',
    '   프록시가 RealImage 생성 (이 시점에 로딩)',
    '   이미지 표시',
    '',
    '3. 이미지 2 표시 요청:',
    '   프록시가 RealImage 생성 (이 시점에 로딩)',
    '   이미지 표시',
    '',
    '4. 이미지 3은 사용 안함:',
    '   프록시만 존재, RealImage 생성 안됨 (메모리 절약)',
    '',
    '✅ 장점: 필요한 것만 로딩하여 성능 최적화',
  ]

  goodOutput.value = results.join('\n')
}

const showBadExample = () => {
  goodOutput.value = ''

  const results = [
    '--- 프록시 미사용 (직접 생성) ---',
    '',
    '1. 초기화 시점:',
    '   RealImage 3개를 모두 즉시 생성 (느림)',
    '   모든 이미지 로딩 완료... (무거운 작업 × 3)',
    '   초기 로딩 시간 3배 증가',
    '   메모리 사용량 증가',
    '',
    '2. 이미지 1 표시 요청:',
    '   이미 로드된 이미지 표시',
    '',
    '3. 이미지 2 표시 요청:',
    '   이미 로드된 이미지 표시',
    '',
    '4. 이미지 3은 사용 안함:',
    '   하지만 이미 로드되어 메모리 차지 중 (낭비)',
    '',
    '❌ 문제점: 사용하지 않을 수도 있는 리소스를',
    '   미리 모두 로딩하여 초기 로딩 시간 증가',
  ]

  badOutput.value = results.join('\n')
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
/* 프록시 패턴 - 시안 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #f0feff 0%, #e0f9ff 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(168, 237, 234, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🩵';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #00b8d4;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(0, 184, 212, 0.1);
}

.description {
  color: #00acc1;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #a8edea;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(168, 237, 234, 0.1);
}

.example-section,
.comparison-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(168, 237, 234, 0.2);
  box-shadow: 0 8px 25px rgba(168, 237, 234, 0.1);
}

.example-section h3,
.comparison-section h3 {
  margin-top: 0;
  color: #26c6da;
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
  background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%);
  color: white;
  box-shadow: 0 6px 20px rgba(168, 237, 234, 0.3);
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(168, 237, 234, 0.4);
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
  background: linear-gradient(135deg, #00b8d4 0%, #0097a7 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(0, 184, 212, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
  max-height: 300px;
  overflow-y: auto;
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
  border: 3px solid rgba(168, 237, 234, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #a8edea;
  margin-top: 0;
  border-bottom: 2px solid #a8edea;
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
