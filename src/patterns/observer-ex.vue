<template>
  <div class="pattern-container">
    <h2>7. 옵저버 패턴 (Observer Pattern)</h2>
    <p class="description">
      한 객체(Subject)의 상태가 변할 때, 그 객체에 의존(구독)하는 다른 객체들(Observers)에게<br />
      자동으로 알림을 보내고 업데이트하는 행위 패턴입니다. (일대다 관계)
    </p>

    <div class="example-section">
      <h3>기상 관측소 시뮬레이터</h3>
      <p>
        '모바일 앱'과 '웹 대시보드'가 기상 관측소의 온도 변화를 구독하고 있습니다.<br />
        온도를 변경하면 구독 중인 모든 장치에 알림이 갑니다.
      </p>
      <div class="button-group">
        <button @click="changeTemperature" class="test-btn">온도 무작위 변경</button>
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
import { ref, reactive, onMounted } from 'vue'

// --- 1. Observer (관찰자 인터페이스) ---
// Subject의 상태 변화에 대한 알림을 받는 update 메서드를 정의합니다.
interface Observer {
  // update 메서드가 로그 문자열을 반환하도록 하여 Vue에서 표시
  update(temperature: number): string
}

// --- 2. Subject (주제 인터페이스) ---
// Observer를 등록, 해지, 알림(notify)하는 메서드를 정의합니다.
interface Subject {
  subscribe(observer: Observer): void
  unsubscribe(observer: Observer): void
  notify(): void
}

// --- 3. ConcreteSubject (구체적인 주제) ---
// Subject 인터페이스를 구현합니다.
// 상태(temperature)를 가지며, 상태가 변경되면 notify()를 호출합니다.
class WeatherStation implements Subject {
  private observers: Observer[] = []
  private temperature: number = 0

  public subscribe(observer: Observer): void {
    if (this.observers.includes(observer)) {
      console.log('[WeatherStation] 이미 구독 중입니다.')
      return
    }
    this.observers.push(observer)
    console.log('[WeatherStation] 새 Observer 구독 시작.')
  }

  public unsubscribe(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer)
    if (observerIndex === -1) {
      console.log('[WeatherStation] 구독 중이 아닌 Observer입니다.')
      return
    }
    this.observers.splice(observerIndex, 1)
    console.log('[WeatherStation] Observer 구독 해지.')
  }

  // 연결된 모든 Observer에게 알림
  public notify(): string[] {
    const logs: string[] = []
    logs.push(`[WeatherStation] ${this.observers.length}개의 Observer에게 알림 전송...`)

    for (const observer of this.observers) {
      // 각 Observer의 update 메서드를 호출
      logs.push(observer.update(this.temperature))
    }
    return logs
  }

  // Subject의 상태를 변경하는 메서드
  // 상태 변경 후 notify()를 호출하여 Observer에게 알림
  public setTemperature(temp: number): string[] {
    this.temperature = temp
    const logs: string[] = []
    logs.push(`--- (기상 관측소 온도가 ${temp}℃로 변경됨) ---`)
    logs.push(...this.notify()) // 알림 전송 및 로그 수집
    return logs
  }
}

// --- 4. ConcreteObserver (구체적인 관찰자) ---
// Observer 인터페이스를 구현합니다.
class DisplayObserver implements Observer {
  private name: string // Observer의 이름 (예: 모바일, 웹)

  constructor(name: string) {
    this.name = name
  }

  // Subject가 notify()를 호출하면 이 메서드가 실행됨
  public update(temperature: number): string {
    const log = `[${this.name}] 알림 수신: 현재 온도는 ${temperature}℃ 입니다.`
    console.log(log)
    return log
  }
}

// --- Vue 로직 ---

const output = ref<string[]>([])

// 1. Subject (기상 관측소) 인스턴스 생성
const weatherStation = new WeatherStation()

onMounted(() => {
  // 2. Observer (디스플레이) 인스턴스 생성
  const mobileDisplay = new DisplayObserver('모바일 앱 📱')
  const webDisplay = new DisplayObserver('웹 대시보드 💻')

  // 3. Subject에 Observer 구독
  weatherStation.subscribe(mobileDisplay)
  weatherStation.subscribe(webDisplay)

  output.value.push('--- 기상 관측소 및 2개의 디스플레이(Observer) 초기화 완료 ---')
  output.value.push('--- (모바일 앱, 웹 대시보드가 관측소를 구독 중) ---')
})

const changeTemperature = () => {
  const newTemp = Math.floor(Math.random() * 40) - 5 // -5 ~ 35도

  // 4. Subject의 상태 변경
  // 이 메서드(setTemperature)가 호출되면
  // 자동으로 notify()가 실행되어 모든 Observer에게 알림이 감.
  const logs = weatherStation.setTemperature(newTemp)
  output.value.push(...logs)
}

const codeExample = `// 1. Observer (관찰자 인터페이스)
interface Observer {
  update(state: string): string; // void가 아닌 string 반환
}

// 2. Subject (주제 인터페이스)
interface Subject {
  subscribe(observer: Observer): void;
  unsubscribe(observer: Observer): void;
  notify(): void;
}

// 3. ConcreteSubject (구체적인 주제)
class WeatherStation implements Subject {
  private observers: Observer[] = [];
  private state: any;

  // ... subscribe, unsubscribe ...

  public notify(): void {
    // 모든 옵저버에게 알림
    for (const observer of this.observers) {
      observer.update(this.state);
    }
  }

  // 상태가 변경되면 알림(notify)을 보냄
  public setState(newState: any): void {
    this.state = newState;
    this.notify();
  }
}

// 4. ConcreteObserver (구체적인 관찰자)
class PhoneDisplay implements Observer {
  public update(state: any): string { // string 반환
    const log = \`[모바일] 날씨 업데이트 받음: \${state}\`;
    console.log(log);
    return log;
  }
}

// --- 사용 ---
const station = new WeatherStation();
const phone = new PhoneDisplay();
const web = new WebDisplay();

// 1. 구독
station.subscribe(phone);
station.subscribe(web);

// 2. 상태 변경 (이때 phone, web의 update가 모두 호출됨)
station.setState("맑음, 25도");
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
  background: #e67e22; /* 옵저버 버튼 색상 변경 */
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
}
.test-btn:hover {
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
