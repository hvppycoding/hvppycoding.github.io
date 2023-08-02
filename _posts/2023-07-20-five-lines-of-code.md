---
title: "파이브 라인스 오브 코드"
excerpt: ""
date: 2023-07-20 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Refactoring
---

## 규칙: 다섯 줄 제한

- **정의**
  - 메서드는 5줄을 초과해서는 안 됩니다.
- **스멜**
  - 메서드가 길다는 것 자체가 스멜입니다.
- **의도**
  - 관심을 가지지 않으면 시간이 지남에 따라 메서드가 커집니다. 각각 5줄의 코드가 있는 4개의 메서드가 20줄인 하나의 메서드보다 훨씬 이해하기 쉽습니다. 각 메서드의 이름으로 코드의 의도를 전달할 수 있기 때문입니다.
- **참조**
  - 메서드 추출 리팩토링

## 리팩터링 패턴: 메서드 추출

- **설명**
  - 한 메서드의 일부를 메서드로 추출합니다.

## 규칙: 호출 또는 전달 중 한 가지만 할 것

- **정의**
  - 함수 내에서는 어떤 객체에 대해 메서드를 호출하거나 다른 함수에 인수로 전달할 수 있습니다. 하지만 둘 다 해서는 안 됩니다.
- **설명**
  - 점점 많은 메서드를 도입하게 되고 여러가지를 파라미터로 전달하게 되면 결국 책임이 고르지 않게 됩니다. 함수의 내용이 동일한 추상화 수준을 유지하는 편이 코드를 읽기가 훨씬 쉽습니다.
- **스멜**
  - 매개변수로 전달한 객체에 대해 옆에 .이 있는지를 확인하여 찾을 수 있습니다. 아래 예시에서 g 객체를 매개변수로 전달하기도 하고 메서드를 호출하기도 합니다.

```typescript
function draw() {
  let canvas = document.getElementById("GameCanvas") as HTMLCanvasElement;
  let g = canvas.getContext("2d");

  g.clearRect(0, 0, canvas.width, canvas.height);   // g의 메서드 호출

  drawMap(g);   // g를 매개변수로 전달
  drawPlayer(g);    // g를 매개변수로 전달
}
```

### 규칙 적용

메서드 추출을 사용해서 이 규칙 위반을 수정해봅시다. `g.clearRect` 줄을 추출하게 되면 `canvas` 객체를 매개변수로 전달해야 됩니다. 그런데 `canvas` 객체에 대해 함수 내에서 `canvas.getContext` 코드가 있어, 다시 `canvas`에 대해서 규칙을 위반하게 됩니다. 대신, 처음의 세 줄을 추출해봅시다. 메서드 추출 시 적절한 이름을 지어 코드를 더 읽기 쉽게 만들 수 있습니다.

```typescript
function draw() {
  let g = createGraphics();
  drawMap(g);
  drawPlayer(g);
}

function createGraphics() {
  let canvas = document.getElementById("GameCanvas") as HTMLCanvasElement;
  let g = canvas.getContext("2d");
  g.clearRect(0, 0, canvas.width, canvas.height);
  return g;
}
```

## 규칙: `if`문은 함수의 시작에만 배치

- **정의**
  - `if`문이 있을 경우 반드시 함수의 첫 줄에 와야합니다.
- **설명**
  - 함수는 한 가지 일만 해야 합니다. 무언가를 확인하는 것은 한 가지 일입니다. 또한 그 후에 다른 것을 하면 안 되므로 유일한 것이어야 합니다.

```typescript
// 변경 전
function reportPrimes(n: number) {
  for (let i = 2; i < n; i++) 
    if (isPrime(i)) 
      console.log(i);
}
```

위 코드는 두 가지 일을 합니다. (1)숫자를 반복합니다. (2)숫자가 소수인지 확인합니다. 따라서 최소한 두 개의 함수가 있어야 합니다.

```typescript
function reportPrimes(n: number) {
  for (let i = 2; i < n; i++) 
    reportIfPrime(i);
}

function reportIfPrime(n: number) {
  if (isPrime(n)) 
    console.log(n);
}
```

## 규칙: `if`문에서 `else`를 사용하지 말 것

- **정의**
  - `if`문에서 `else`를 사용해서는 안 됩니다. (다룰 수 없는 data type 체크하는 경우는 예외)
- **설명**
  - 결정을 내리는 것은 어렵습니다. `if-else`를 사요하게 되면 결정을 내리는 지점이 고정되게 됩니다. 한 번 결정된 `if-else`에 대해서 나중에 변경이 불가능하기 때문에 코드의 유연성이 떨어지게 됩니다. `if-else`는 하드코딩된 결정이라고 볼 수 있습니다. 코드에서 하드코딩이 좋지 않은 것처럼 하드코딩된 결정도 좋지 않습니다. 독립된 `if`문은 *check*로 간주하고, `if-else`를 *decision*이라고 생각합시다.

```typescript
// 변경 전
function average(ar: number[]) {
  if (size(ar) === 0)
    throw "Empty array not allowed";
  else
    return sum(ar) / size(ar);
}

// 변경 후
function assertNotEmpty(ar: number[]) {
  if (size(ar) === 0)
    throw "Empty array not allowed";
}
function average(ar: number[]) {
  assertNotEmpty(ar);  
  return sum(ar) / size(ar);
}
```

## 리팩터링 패턴: 클래스로 타입 코드 대체

- **설명**
  - 열거형을 인터페이스로 변환하고 열거형들의 각 값은 클래스가 됩니다. 그렇게하면 각 값에 속성을 추가하고 관련 기능을 만들 수 있게 됩니다. 또한, 추후에 열거형에 대한 `switch` 혹은 `else if` 체인을 제거할 수 있게 해줍니다.

```typescript
// 초기 코드
enum TrafficLight {
  RED, YELLOW, GREEN
}
function updateCarForLight(current: TrafficLight) {
  if (current === TrafficLight.RED)
    car.stop();
  else
    car.drive();
}
```

```typescript
// 변경 후
interface TrafficLight {
  isRed(): boolean;
  isYellow(): boolean;
  isGreen(): boolean;
}

class Red implements TrafficLight {
  isRed() { return true; }
  isYellow() { return false; }
  isGreen() { return false; }
}

class Yellow implements TrafficLight {
  isRed() { return false; }
  isYellow() { return true; }
  isGreen() { return false; }
}

class Green implements TrafficLight {
  isRed() { return false; }
  isYellow() { return false; }
  isGreen() { return true; }
}

function updateCarForLight(current: TrafficLight) {
  if (current.isRed())
    car.stop();
  else
    car.drive();
}
```

## 리팩터링 패턴: 클래스로의 코드 이관

- **설명**
  - 기능을 클래스로 옮겨, 결과적으로 `if` 구문이 제거되고 기능이 데이터에 가까이 이동합니다.

```typescript
// 변경 후
interface TrafficLight {
  isRed(): boolean;
  isYellow(): boolean;
  isGreen(): boolean;
  updateCar(): void;
}

class Red implements TrafficLight {
  isRed() { return true; }
  isYellow() { return false; }
  isGreen() { return false; }
  updateCar() { car.stop(); }
}

class Yellow implements TrafficLight {
  isRed() { return false; }
  isYellow() { return true; }
  isGreen() { return false; }
  updateCar() { car.drive(); }
}

class Green implements TrafficLight {
  isRed() { return false; }
  isYellow() { return false; }
  isGreen() { return true; }
  updateCar() { car.drive(); }
}

function updateCarForLight(current: TrafficLight) {
  current.updateCar();
}
```

## 리팩터링 패턴: 메서드의 인라인화

- **설명**
  - 프로그램에서 더 이상 가독성에 도움이 되지 않는 메서드를 제거합니다. 그 방법은 메서드의 코드를 이를 호출하는 모든 곳으로 옮기는 것입니다.

## 리팩터링 패턴: 메서드 전문화

- **설명**
  - 프로그래머들은 일반화하고 재사용하려는 본능적인 욕구가 있지만 그렇게 하면 책임이 흐려지고 다양한 위치에서 코드를 호출할 수 있기 때문에 오히려 문제가 될 수도 있습니다.

## 규칙: `switch`를 사용하지 말 것

- **정의**
  - `default` 케이스가 없고 모든 case에 대한 반환 값이 있는 경우가 아니라면 `switch`를 사용하지 마십시오.
- **설명**
  - `switch`를 사용할 경우 무엇을 처리할지와 무엇을 처리하지 않을지는 불변속성이 됩니다. 또한 `break` 키워드를 만나기 전까지 케이스를 연속해서 실행하는 폴스루(fall-through) 로직이라는 점입니다. `switch`를 사용할 때 `break` 키워드를 쓰는 것을 누락하기 쉽습니다.

## 규칙: 인터페이스에서만 상속받을 것

- **정의**
  - 상속은 오직 인터페이스를 통해서만 받습니다.
- **설명**
  - 사람들이 추상 클래스를 사용하는 가장 일반적인 이유는 일부 메서드에는 기본 구현을 제공하고 다른 메서드는 추상화하기 위한 것입니다. 이것은 중복을 줄이고 코드의 줄을 줄이고자 할 경우 편리합니다. **그러나 안타깝게도 단점이 훨씬 더 많습니다.** 코드 공유는 커플링(결합)을 유발합니다. 기본 구현된 메서드가 있는 경우 다음의 두 가지 시나리오가 있습니다. (1) 가능한 모든 하위 클래스에 해당 메서드가 필요한 경우, 이 경우는 메서드를 클래스 밖으로 쉽게 이동할 수 있습니다. (2) 일부 하위 클래스에서 메서드를 재정의해야 하는 경우, 기본 구현이 이미 있기 때문에 컴파일러를 통해 재정의가 필요한 메서드인지 아닌지 알아낼 수가 없습니다. 이 경우를 명시적으로 처리할 필요가 있기 때문에 완전히 추상화된 메서드로 남겨두는 것이 좋습니다. 여러 클래스에서 코드를 공유해야 하는 경우, 해당 코드를 다른 공유 클래스에 넣을 수 있습니다(전략 패턴 참고).

## 규칙: 순수 조건 사용

- **정의**
  - 조건문은 항상 순수 조건문이어야 합니다.
- **설명**
  - 순수(pure)라는 말은 조건에 부수적인 동작이 없음을 의미합니다. 부수적인 동작이란 조건이 변수에 값을 할당하거나 예외를 발생시키거나 출력, 파일 쓰기 등과 같이 I/O와 상호작용하는 것을 의미합니다.

## 리팩터링 패턴: 전략 패턴의 도입

- **설명**
  - 다른 클래스를 인스턴스화해서 변형을 도입하는 개념을 전략 패턴이라고 합니다.

```typescript
// 변경 전
class ArrayMinimum {
  constructor(private accumulator: number) {
  }
  process(arr: number[]) {
    for(let i = 0; i < arr.length; i++) {
      if (this.accumulator > arr[i])
        this.accumulator = arr[i];
    }
    return this.accumulator;
  }
}

class ArraySum {
  constructor(private accumulator: number) {
  }
  process(arr: number[]) {
    for(let i = 0; i < arr.length; i++) {
      if (this.accumulator > arr[i])
        this.accumulator += arr[i];
    }
    return this.accumulator;
  }
}
```

```typescript
// 변경 후
class BatchProcessor {
  constructor(private processor: ElementProcessor) { }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processor.process(arr[i]);
    }
  }
}

interface ElementProcessor {
  processElement(e: number): void;
  getAccumulator(): number;
}

class MinimumProcessor implements ElementProcessor {
  constructor(private accumulator: number) { }
  getAccumulator() { return this.accumulator; }
  processElement(e: number) {
    if (this.accumulator > e)
      this.accumulator = e;
  }
}

class SumProcessor implements ElementProcessor {
  constructor(private accumulator: number) { }
  getAccumulator() { return this.accumulator; }
  processElement(e: number) {
    this.accumulator += e;
  }
}
```

## 규칙: 구현체가 하나뿐인 인터페이스를 만들지 말 것

## 규칙: getter와 setter를 사용하지 말 것

- **정의**
  - 부울(Boolean)이 아닌 필드에 setter나 getter를 사용하지 마십시오.
- **스멜**
  - '낯선 사람에게 말하지 말라'로 요약되는 디미터 법칙에서 유래했습니다. 여기서 낯선 사람이란 우리가 직접 접근할 수는 없지만 참조를 얻을 수는 있는 객체입니다.
- **의도**
  - 푸시 기반 아키텍처에서는 서비스와 같은 메서드를 노출합니다. 이런 메서드의 사용자는 사용하는 메서드의 내부 구조에 대해 신경 쓰지 않아도 됩니다.

## 리팩터링 패턴: getter와 setter 제거하기

- **설명**
  - 기능을 데이터에 더 가깝게 이동하여 getter와 setter를 제거할 수 있습니다.
