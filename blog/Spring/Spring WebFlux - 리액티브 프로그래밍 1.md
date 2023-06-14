태그: #Spring 

# 리액티브 프로그래밍

## 리액티브 프로그래밍이란?

### 리액티브 시스템(Reactive System)
- 리액티브 시스템은 한마디로 반응을 잘하는 시스템을 의미하는 것으로, 여기서의 반응은 리액티브 시스템을 이용하는 클라이언트의 요청에 반응을 잘하는 시스템을 의미함
- 리액티브 시스템 관점에서의 반응은 스레드의 Non-Blocking과 관련이 있음
- 리액티브 시스템은 클라이언트의 요청에 대한 응답 대기 시간을 최소화할 수 있도록 요청 스레드가 차단되지 않게 함으로써(Non-Blocking) 클라이언트에게 즉각적으로 반응하도록 구성된 시스템이라고 볼 수 있음

<br>

- 리액티브 시스템의 특징
<br>

![](image_38.png)

<br>

- MEANS
    - 리액티브 시스템에서 사용하는 커뮤니케이션 수단
    - Message Driven
        - 리액티브 시스템에서는 메시지 기반 통신을 통해 여러 시스템 간에 느슨한 결합을 유지함
- FORM
    - 메시지 기반 통신을 통해 리액티브 시스템이 어떤 특성을 가지는 구조로 형성되는지를 의미함
    - Elastic
        - 시스템으로 들어오는 요청량이 적거나 많거나에 상관없이 일정한 응답성을 유지하는 것을 의미함
    - Resillient
        - 시스템의 일부분에 장애가 발생하더라도 응답성을 유지하는 것을 의미함
- VALUE
    - 리액티브 시스템의 핵심 가치가 무엇인지를 표현하는 영역
    - Responsive
        - 리액티브 시스템은 클라이언트의 요청에 즉각적으로 응답할 수 있어야 함을 의미함
    - Maintainable
        - 클라이언트의 요청에 대한 즉각적인 응답이 지속가능해야 함을 의미함
    - Extensible
        - 클라이언트의 요청에 대한 처리량을 자동으로 확장하고 축소할 수 있어야 함을 의미함

<br>

### 리액티브 프로그래밍(Reactive Programming)
- 리액티브 프로그래밍은 한마디로 리액티브 시스템에서 사용되는 프로그래밍 모델을 의미함
- 리액티브 시스템에서의 메시지 기반 통신(Message Driven)은 Non-Blocking 통신과 유기적인 관계를 맺고 있으며, 리액티브 프로그래밍은 Non-Blocking 통신을 위한 프로그래밍 모델

<br>

#### 리액티브 프로그래밍의 특징
- declarative programming paradigm
    - 선언형 프로그래밍 방식을 사용하는 대표적인 프로그래밍 모델
- data streams and the propagation of change
    - data streams는 지속적으로 데이터가 입력으로 들어올 수 있음을 의미하며, 리액티브 프로그래밍에서는 데이터가 지속적으로 발생하는 것 자체를 데이터에 어떤 변경이 발생함을 의미함
    - 이 변경 자체를 이벤트로 간주하고, 이벤트가 발생할 때마다 데이터를 계속해서 전달함
- automatic propagation of the changed data flow
    - data streams and **the propagation of change**와 같은 의미함
    - 지속적으로 발생하는 데이터를 하나의 데이터 플로우로 보고 데이터를 자동으로 전달함

<br>

### 리액티브 스트림즈(Reactive Streams)
- 리액티브 프로그래밍을 위한 표준 사양(또는 명세, Specification)

<br>

#### 리액티브 스트림즈 컴포넌트
- Publisher
    - Publisher 인터페이스는 데이터 소스로 부터 데이터를 내보내는(emit) 역할을 함
    - Publisher 인터페이스는 subscribe() 추상 메서드를 포함하고 있으며, subscribe()의 파라미터로 전달되는 Subscriber가 Publisher로부터 내보내진 데이터를 소비하는 역할을 함
    - subscribe()는 메서드 이름에서도 알 수 있듯이 Publisher가 내보내는 데이터를 수신할지 여부를 결정하는 구독의 의미를 가지고 있으며, 일반적으로 subscribe()가 호출되지 않으면 Publisher가 데이터를 내보내는 프로세스는 시작되지 않
    - Publisher가 데이터를 내보내는 것을 'emit한다' 라고 표현함
<br>

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

<br>

- Subscriber
    - Subscriber 인터페이스는 Publisher로부터 내보내진 데이터를 소비하는 역할을 함
    - Subscriber는 네 개의 추상 메서드를 포함하고 있음
        - onSubscribe(Subscription s)
            - 구독이 시작되는 시점에 호출되며, onSubscribe() 내에서 Publisher에게 요청할 데이터의 개수를 지정하거나 구독 해지 처리를 할 수 있음
        - onNext(T t)
            - Publisher가 데이터를 emit할 때 호출되며, emit 된 데이터를 전달받아서 소비할 수 있음
        - onError(Throwable t)
            - Publisher로부터 emit 된 데이터가 Subscriber에게 전달되는 과정에서 에러가 발생할 경우에 호출됨
        - onComplete()
            - Publisher가 데이터를 emit하는 과정이 종료될 경우 호출되며, 데이터의 emit이 정상적으로 완료된 후, 처리해야 될 작업이 있다면 onComplete() 내에서 수행할 수 있음
<br>

```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

<br>

- Subscription
    - Subscription 인터페이스는 Subscriber의 구독 자체를 표현한 인터페이스
        - request(long n)
            - Publihser가 emit하는 데이터의 개수를 요청함
        - cancel()
            - 구독을 해지하는 역할을 함
            - 구독 해지가 발생하면 Publisher는 더 이상 데이터를 emit하지 않음
<br>

```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```

<br>

- Processor
    - Processor 인터페이스는 Subscriber 인터페이스와 Publisher 인터페이스를 상속하고 있기 때문에 Publisher와 Subscriber의 역할을 동시에 할 수 있는 특징을 가지고 있으며, 별도로 구현해야 되는 추상 메서드는 없음
<br>

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

<br>

### 리액티브 스트림즈의 구현체
- Project Reactor
    - Project Reactor(줄여서 Reactor)는 리액티브 스트림즈를 구현한 대표적인 구현체로써 Spring과 궁합이 가장 잘 맞는 리액티브 스트림즈 구현체
    - Reactor는 Spring 5의 리액티브 스택에 포함되어 있으며 Sprig Reactive Application 구현에 있어 핵심적인 역할을 담당하고 있음
- RxJava
    - RxJava는 .NET 기반의 리액티브 라이브러리를 넷플릭스에서 Java 언어로 포팅한 JVM 기반의 리액티브 확장 라이브러리
    - RxJava의 경우 2.0부터 리액티브 스트림즈 표준 사양을 준수하고 있으며, 이 전 버전의 컴포넌트와 함께 혼용되어 사용이 되고 있음
- Java Flow API
    - Java는 Java 9부터 리액티브 스트림즈를 지원하고 있음
    - Flow API는 리액티브 스트림즈를 구현한 구현체가 아니라 리액티브 스트림즈 표준 사양을 Java 안에 포함을 시킨 구조라고 볼 수 있음
    - 다양한 벤더들이 JDBC API를 구현한 드라이버를 제공할 수 있도록 SPI(Service Provider Interface) 역할을 하는 것처럼 Flow API 역시 리액티브 스트림즈 사양을 구현한 여러 구현체들에 대한 SPI 역할을 함
- 기타 리액티브 확장(Reactive Extension)
    - RxJava의 Rx는 Reactive Extension의 줄임말로, 특정 언어에서 리액티브 스트림즈를 구현한 별도의 구현체가 존재한다는 의미이며, 실제로 다양한 프로그래밍 언어에서 리액티브 스트림즈를 구현한 리액티브 확장(Reactive Extension) 라이브러리를 제공하고 있음
    - 대표적인 리액티브 확장(Reactive Extension) 라이브러리로 앞에서 언급한 RxJava가 있으며, 이외에도 RxJS, RxAndroid, RxKotlin, RxPython, RxScala 등이 있음