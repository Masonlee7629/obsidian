태그: #Spring 

# Project Reactor

## Project Reactor란?

### Reactor란?
- Reactor는 리액티브 스트림즈 표준 사양을 구현한 구현체 중 하나
- Reactor는 Spring 5 버전부터 지원하는 리액티브 스택에 포함되어 리액티브 한 애플리케이션으로 동작하는 데 있어 핵심적인 역할을 담당하는 리액티브 프로그래밍을 위한 라이브러리

<br>

### Reactor 특징
1. Reactor는 리액티브 스트림즈(Reactive Streams)를 구현한 리액티브 라이브러리
2. 가장 많이 나오는 용어가 바로 Non-Blocking으로, Non-Blocking은 리액티브 프로그래밍의 핵심적인 특징이며, Reactor 역시 완전한 Non-Blocking 통신을 지원함
3. Reactor는 Publisher 타입으로 Mono[0|1]와 Flux[N]이라는 두 가지 타입을 제공
    - Mono[0|1]에서 0과 1의 의미는 0건 또는 1건의 데이터를 emit 할 수 있음을 의미함
    - Flux[N]에서 N의 의미는 여러 건의 데이터를 emit할 수 있음을 의미함
4. 서비스들 간의 통신이 잦은 MSA(Microservice Architecture) 기반 애플리케이션들은 요청 스레드가 차단되는 Blocking 통신을 사용하기에는 무리가 있으므로, 기본적으로 Non-Blocking 통신을 완벽하게 지원하는 Reactor는 MSA 구조에 적합한 라이브러리라고 볼 수 있음
5. Backpressure란 Subscriber의 처리 속도가 Publihser의 emit 속도를 따라가지 못할 때 적절하게 제어하는 전략을 의미함
    - Publisher에서 끊임없이 들어오는 데이터를 emit하는 것과 달리 Subscriber의 처리 속도가 느리면 오버플로우가 발생하게 되고 급기야는 시스템이 다운될 수 있음

<br>

### Hello Reactor로 알아보는 Reactor 구성 요소

```java
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;

public class HelloReactorExample {
    public static void main(String[] args) throws InterruptedException {
        Flux    // (1)
            .just("Hello", "Reactor")               // (2)
            .map(message -> message.toUpperCase())  // (3)
            .publishOn(Schedulers.parallel())       // (4)
            .subscribe(System.out::println,         // (5)
                    error -> System.out.println(error.getMessage()),  // (6)
                    () -> System.out.println("# onComplete"));        // (7)

        Thread.sleep(100L);
    }
}

// 출력 결과
// HELLO
// REACTOR
// # onComplete
```

<br>

- (1)은 Reactor Sequence의 시작점으로, (1)에서 Flux로 시작한다는 것은 Reactor Sequence가 여러 건의 데이터를 처리함을 의미함
- (2) just() Operator는 원본 데이터 소스로부터(Original Data Source) 데이터를 emit하는 Publisher의 역할을 함
- (3) map() Operator는 Publisher로부터 전달받은 데이터를 가공하는 Operator로, (3)에서는 just() Operator에서 emit 된 영문 문자열을 대문자로 변환하고 있음
    - Reactor에서는 map() Operator처럼 데이터를 가공 처리할 수 있는 수많은 Operator를 지원함
- (4) publishOn() Operator는 Reactor Sequence에서 스레드 관리자 역할을 하는 Scheduler를 지정하는 Operator
    - (4)와 같이 publishOn() Operator에 Scheduler를 지정하면 publishOn()을 기준으로 Downstream의 실행 스레드가 Scheduler에서 지정한 유형의 스레드로 변경함
    - 위 코드에서는 Reactor Sequence 상에서 두 개의 스레드가 실행됨
- subscribe()는 파라미터로 총 세 개의 람다 표현식을 가지는데 (5)의 첫 번째 파라미터는 Publisher가 emit한 데이터를 전달받아서 처리하는 역할을 함
- (6)의 두 번째 파라미터는 Reqctor Sequence 상에서 에러가 발생할 경우, 해당 에러를 전달받아서 처리하는 역할을 함
- (7)의 세 번째 파라미터는 Reactor Sequence가 종료된 후 어떤 후 처리를 하는 역할을 함
- 출력 결과를 보면 두 개의 문자열이 map() Operator를 거쳐 대문자로 변환된 후, subscribe()의 첫 번째 람다 표현식으로 전달되어, 출력됨
- “# onComplete” 문자열이 출력되었다는 것은 Reactor의 Sequence가 정상적으로 종료되었다는 의미함
    - subscribe()의 세 번째 파라미터인 람다 표현식은 Reactor Sequence가 정상적으로 종료되면 동작을 수행한다는 사실을 알 수 있음
- Reactor Sequence에 Scheduler를 지정하면 main 스레드 이외에 별도의 스레드가 하나 더 생기고, Reactor에서 Scheduler로 지정한 스레드는 모두 데몬 스레드이기 때문에 주 스레드인 main 스레드가 종료되면 동시에 종료됨
    - main 스레드를 Thread.sleep(100L)을 통해 0.1초 정도 동작을 지연시키면 그 0.1초 사이에 Scheduler로 지정한 데몬 스레드를 통해 Reactor Sequence가 정상 동작을 하게 됨

<br>

---

<br>

## 마블 다이어그램(Marble Diagram)

### 마블 다이어그램이란?
- 마블(Marble)은 실제로 구슬이라는 뜻으로, 구슬 모양의 알록달록한 동그라미는 하나의 데이터를 의미하며, 다이어그램 상에서 시간의 흐름에 따라 변화하는 데이터의 흐름을 표현함
- 리액티브 프로그래밍에서는 어떤 Operator를 사용하느냐에 따라서 데이터의 흐름이 다양하게 변화할 수 있으며, 이러한 복잡한 데이터의 흐름을 마블 다이어그램을 통해 좀 더 쉽게 이해할 수 있음
- Reactor에서 마블 다이어그램을 보는 제일 중요한 이유는 Reactor에서 제공하는 수많은 Operator의 내부 동작을 조금 더 쉽게 이해하고, 해당 Operator를 적절하게 사용하기 위함

<br>

### 마블 다이어그램(Marble Diagram)으로 Mono와 Flux 이해하기

#### Mono의 마블 다이어그램

![](image_39.png)

<br>

- 마블 다이어그램에는 아래위로 두 개의 타임 라인이 있는데 모두 데이터가 흘러가는 시간의 흐름을 표현하고 있으며, 시간은 왼쪽에서 오른쪽으로 흘러가기 때문에 시간 상으로는 왼쪽이 빠른 시간을 의미함
- (1)은 원본 Mono(Original Mono)에서 Sequence가 시작되는 것을 타임라인으로 표현한 것
- (2)는 Mono의 Sequence에서 데이터가 emit 되는 것을 표현하고 있음
    - 그림 상에서 구슬 모양의 데이터 하나가 emit되는 것을 확인할 수 있는데, 단순히 그냥 구슬 모양의 데이터 하나만 표시한 게 아니라 Mono는 0건 또는 1건의 데이터만 emit하는 Reactor 타입이기 때문에 마블 다이어그램에서 이를 표현하고 있는 것
- (3)의 수직 막대 바는 Mono의 Sequence가 정상 종료됨을 의미함
- (4)는 Mono에서 지원하는 어떤 Operator에서 입력으로 들어오는 구슬 모양의 데이터를 가공 처리되는 것을 표현하고 있음
- (5)는 Operator에서 가공 처리된 데이터가 Downstream으로 전달될 때의 타임라인이며, 시간의 흐름은 위쪽에 있는 타임라인과 마찬가지로 왼쪽이 빠른 시간
- 만약 Mono에서 emit 된 데이터가 처리되는 과정에 에러가 발생한다면 (6)과 같이 ‘X’로 표시함
    - ‘|’는 정상 종료, ‘X’는 에러로 인한 비정상 종료를 의미함

<br>

#### Flux의 마블 다이어그램

![](image_40.png)

<br>

- Flux의 마블 다이어그램은 앞에서 살펴본 Mono의 마블 다이어그램과 큰 차이가 없으나 (2)와 같이 emit되는 데이터가 Mono와는 달리 구슬이 하나가 아니라 여러 개의 구슬이 있음
- Mono가 0 또는 1개의 데이터만 emit하는 것과는 달리 Flux는 여러 개(0 … N)의 데이터를 emit하는 Reactor 타입임을 표현하는 것

<br>

#### Operator의 마블 다이어그램 예시

![](image_41.png)

<br>

- map() Operator는 입력으로 들어오는 데이터를 개발자가 구현하는 동작대로 변환해서 Downstream으로 전달하는 역할을 함
- 색깔을 가진 동그라미 데이터를 동일한 색의 네모로 변환해서 Downstream으로 전달하고 있음

<br>

```java
// map() Operator의 사용 예
import reactor.core.publisher.Flux;

public class MarbleDiagramExample {
    public static void main(String[] args) {
        Flux
            .just("Green-Circle", "Orange-Circle", "Blue-Circle")   // (1)
            .map(figure -> figure.replace("Circle", "Rectangle"))   // (2)
            .subscribe(System.out::println);   // (3)
    }
}
```

- (1)에서 세 개의 문자열(색깔을 가지는 동그라미를 문자열로 표현)을 emit함
- (2)를 보면 map() Operator 내부에서 동그라미를 네모로 바꾸고 있음
- map() Operator 내부에서 변환된 문자열을 (3)에서 출력함

<br>

---

<br>

## 스케줄러(Scheduler)

### 스케줄러란?
- Scheduler는 한마디로 Reactor Sequence 상에서 처리되는 동작들을 하나 이상의 스레드에서 동작하도록 별도의 스레드를 제공해준다고 할 수 있음
- Reactor는 기본적으로 Non-Blocking 통신을 위한 비동기프로그래밍을 위해 탄생했기 때문에 여러 스레드를 손쉽게 관리해 주는 Scheduler는 Reactor에서 중요한 역할을 함
- Java의 멀티스레딩 프로그래밍은 굉장히 복잡하데 Reactor의 Scheduler를 사용하면 멀티스레딩에서 발생할 수 있는 여러 문제점들에 대비하기 위한 복잡한 처리를 직접 할 필요가 없음
- Reactor의 Scheduler는 복잡한 멀티스레딩 프로세스를 단순하게 해 줌

<br>

### Scheduler 전용 Operator
- Reactor에서는 Scheduler를 위한 별도의 Operator를 제공함
- 적절한 상황에 맞는 스레드를 추가로 생성하는 Operator
<br>

```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

/**
 * Scheduler를 추가하지 않을 경우
 */
@Slf4j
public class SchedulersExample01 {
    public static void main(String[] args) {
        Flux
            .range(1, 10)
            .filter(n -> n % 2 == 0)
            .map(n -> n * 2)
            .subscribe(data -> log.info("# onNext: {}", data));
    }
}
```

<br>

- range() Operator를 이용해 1부터 10개의 숫자를 emit한 후, emit 된 숫자 데이터 중에서 filter() Opertor를 이용해 짝수만 필터링 한 뒤, 필터링된 데이터를 map() Operator에서 2를 곱해서 Subscriber에게 전달하는 간단한 예제
- Scheduler를 지정하지 않았기 때문에 실행 결과를 보면 main 스레드에서 실행이 되는 것을 확인할 수 있음

<br>

#### subscribeOn() Operator

```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;

/**
 * subscribeOn() Operator를 이용해서 Scheduler를 추가할 경우
 */
@Slf4j
public class SchedulersExample02 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .range(1, 10)
            .doOnSubscribe(subscription -> log.info("# doOnSubscribe"))   // (1)
            .subscribeOn(Schedulers.boundedElastic())     // (2)
            .filter(n -> n % 2 == 0)
            .map(n -> n * 2)
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(100L);
    }
}
```

<br>

- subscribeOn()이라는 Operator를 추가해서 스레드를 하나 더 생성함
- (2)와 같이 subscribeOn() Operator 내부에 Schedulers.boundedElastic() 같은 Scheduler를 지정하면 구독 직후에 실행되는 스레드가 main 스레드에서 Scheduler로 지정한 스레드로 바뀌게 됨
- subscribeOn()의 경우, 구독 직후 실행되는 Operator 체인의 실행 스레드를 Scheduler에서 지정한 스레드로 변경함
- doOnSubscribe() Operator는 구독 발생 직후에 트리거 되는 Operator로써 구독 직후에 실행되는 스레드와 동일한 스레드에서 실행됨
    - 만약 구독 직후에 어떤 동작을 수행하고 싶다면 doOnSubscribe()에 로직을 작성하면 됨

<br>

#### publishOn() Operator

```java
@Slf4j
public class SchedulersExample03 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .range(1, 10)
            .subscribeOn(Schedulers.boundedElastic())
            .doOnSubscribe(subscription -> log.info("# doOnSubscribe"))

            .publishOn(Schedulers.parallel())  // (1)
            .filter(n -> n % 2 == 0)
            .doOnNext(data -> log.info("# filter doOnNext"))  // (2)

            .publishOn(Schedulers.parallel())    // (3)
            .map(n -> n * 2)
            .doOnNext(data -> log.info("# map doOnNext")) // (4)

            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(100L);
    }
}
```

<br>

- subscribeOn()이라는 Operator를 추가한 상태에서, publishOn()이라는 Operator를 추가함
- (1), (3)과 같이 두 개의 publishOn()이 추가함
- publishOn()을 추가할 때마다 추가한 publishOn()을 기준으로 Downstream 쪽 스레드가 publishOn()에서 Scheduler로 지정한 스레드로 변경됨
- (2)와 (4)의 doOnNext()는 doOnNext() 바로 앞에 위치한 Operator가 실행될 때, 트리거 되는 Operator로 여기서는 filter()와 map() Operator가 어느 스레드에서 실행이 되는지 확인하기 위한 용도로 사용하고 있음
- publishOn() Operator가 fiter()와 map() Operator 앞에 추가되면서 publishOn()이 추가될 때마다 실행 스레드가 바뀌는 것을 확인할 수 있음

<br>

- subscribeOn()과 publishOn() 정리
    - subscribeOn()은 구독 시점 직후의 실행 흐름을 다른 스레드로 바꾸는 데 사용함
        - range() Operator처럼 **원본 데이터를 생성하고, 생성한 데이터를 emit하는 작업**이 구독 직후에 실행 됨
        - subscribeOn()은 주로 데이터 소스에서 데이터를 emit하는 원본 Publisher의 실행 스레드를 지정하는 역할을 함
    - publishOn()은 전달받은 데이터를 가공 처리하는 Operator 앞에 추가해서 실행 스레드를 별도로 추가하는 역할을 함
        - publishOn()은 Operator 앞에 여러 번 추가할 경우 별도의 스레드가 추가로 생성되지만 subscribeOn()은 여러 번 추가해도 하나의 스레드만 추가로 생성 됨
    - 대부분의 요청은 적은 수의 스레드로 모두 처리가 되지만 스레드가 직접 복잡한 계산을 수행할 경우에는 응답 처리 시간이 늦어질 수 있움
        - 복잡한 계산이 필요한 작업의 경우 서버 엔진에서 생성하는 요청 처리 스레드가 복잡한 계산 작업으로 인해 응답 지연이 발생하지 않도록 별도의 스레드가 필요할 수 있음
        - 이 경우 Scheduler를 통해 별도의 스레드를 생성한 후, 복잡한 계산을 수행하도록 할 수 있음
    - Reactor에서는 Scheduler를 통해 여러 가지 유형의 스레드를 지원함
        - subscribeOn()에서는 주로 Schedulers.&#42;boundedElastic&#42;()을 사용하고, publishOn()에서는 주로 Schedulers.&#42;parallel&#42;()을 사용함