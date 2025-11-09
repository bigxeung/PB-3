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
/* 옵저버 패턴 - 로즈 그라디언트 테마 */
.pattern-container {
  padding: 2.5rem;
  max-width: 950px;
  margin: 20px auto;
  background: linear-gradient(145deg, #fff0f3 0%, #ffe4e9 100%);
  border: 4px solid transparent;
  border-radius: 30px;
  box-shadow: 0 10px 40px rgba(255, 154, 158, 0.15);
  position: relative;
  overflow: hidden;
}

.pattern-container::before {
  content: '🌹';
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 3rem;
  opacity: 0.3;
}

h2 {
  color: #ff6b81;
  margin-bottom: 0.8rem;
  font-size: 2rem;
  font-weight: 800;
  text-shadow: 2px 2px 4px rgba(255, 107, 129, 0.1);
}

.description {
  color: #ee5a6f;
  margin-bottom: 2rem;
  line-height: 1.8;
  background: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 20px;
  border-left: 5px solid #ff9a9e;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(255, 154, 158, 0.1);
}

.example-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  border: 3px solid rgba(255, 154, 158, 0.2);
  box-shadow: 0 8px 25px rgba(255, 154, 158, 0.1);
}

.example-section h3 {
  margin-top: 0;
  color: #ff7979;
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
  background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 6px 20px rgba(255, 154, 158, 0.3);
  transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  margin-top: 0.5rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}
.test-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 30px rgba(255, 154, 158, 0.4);
}
.test-btn:active {
  transform: translateY(0) scale(0.98);
}

.output {
  margin-top: 1.5rem;
  background: linear-gradient(135deg, #ff6b81 0%, #ff8fab 100%);
  color: #fff;
  padding: 1.5rem;
  border-radius: 20px;
  white-space: pre-wrap;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 15px;
  line-height: 1.6;
  box-shadow: 0 8px 25px rgba(255, 107, 129, 0.3);
  border: 3px solid rgba(255, 255, 255, 0.2);
  max-height: 300px;
  overflow-y: auto;
}

.code-section {
  background: linear-gradient(135deg, #1e1e2e 0%, #2a2a3e 100%);
  padding: 2rem;
  border-radius: 25px;
  overflow-x: auto;
  border: 3px solid rgba(255, 154, 158, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
}

.code-section h4 {
  color: #ff9a9e;
  margin-top: 0;
  border-bottom: 2px solid #ff9a9e;
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
