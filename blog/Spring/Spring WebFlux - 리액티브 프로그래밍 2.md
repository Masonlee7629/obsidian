태그: #Spring 
연결문서: [Spring WebFlux - Project Reactor 1](Spring%20WebFlux%20-%20Project%20Reactor%201.md)

# 리액티브 프로그래밍

## 리액티브 프로그래밍 구조

### 명령형 프로그래밍 vs 선언형 프로그래밍

#### 명령형 프로그래밍

```java
public class ImperativeProgrammingExample {
    public static void main(String[] args){
        // List에 있는 숫자들 중에서 4보다 큰 짝수의 합계 구하기
        List<Integer> numbers = List.of(1, 3, 6, 7, 8, 11);
        int sum = 0;

        for(int number : numbers){
            if(number > 4 && (number % 2 == 0)){
                sum += number;
            }
        }

        System.out.println(sum);
    }
}
```

<br>

- List에 포함된 숫자들을 for문을 이용해서 순차적으로 접근한 후, if 문으로 특정 조건에 맞는 숫자들만 sum 변수에 더해서 합계를 구하고 있음
- 명령형 프로그래밍 방식의 경우, 코드가 어떤 식으로 실행되어야 하는지에 대한 구체적인 로직들이 코드 안에 그대로 드러남

<br>

#### 선언형 프로그래밍

```java
public class DeclarativeProgramingExample {
    public static void main(String[] args){
        // List에 있는 숫자들 중에서 4보다 큰 짝수의 합계 구하기
        List<Integer> numbers = List.of(1, 3, 6, 7, 8, 11);

        int sum =
                numbers.stream()
                        .filter(number -> number > 4 && (number % 2 == 0))
                        .mapToInt(number -> number)
                        .sum();

        System.out.println("# 선언형 프로그래밍: " + sum);
    }
}
```

<br>

- Java에서 선언형 프로그래밍 방식을 이해하기 위한 가장 적절한 예는 Java 8부터 지원하는 Stream API
- List에 포함된 숫자들을 처리하는 것 명령형 프로그래밍과 동일하지만 처리 방식은 전혀 다름
- Java Stream API를 사용하기 때문에 코드 상에 보이지 않는 내부 반복자가 명령형 프로그래밍 방식에서 사용하는 for문을 대체하고 있고, filter() 메서드(Operation)가 if 문을 대신해서 조건에 만족하는 숫자를 필터링하고 있음
- Stream API를 사용한 코드는 numbers.stream().filter().mpaToInt().sum()과 같은 메서드 체인이 순차적으로 실행이 되는 것이 아님
- 선언형 프로그래밍 방식은 하나부터 열까지 개발자가 일일이 로직을 모두 작성하지 않고 정말 필요한 동작들을 람다 표현식으로 정의(선언)하고 구체적인 동작 수행은 Operation(연산) 메서드 체인에 위임함

<br>

### 리액티브 프로그래밍의 구조

```java
import reactor.core.publisher.Mono;

// 리액티브 프로그래밍 기본 구조
public class HelloReactiveExample01 {
    public static void main(String[] args) {
        // (1) Publisher의 역할
        Mono<String> mono = Mono.just("Hello, Reactive");

        // (2) Subscriber의 역할
        mono.subscribe(message -> System.out.println(message));
    }
}
```

<br>

- 새로운 프로그래밍 언어를 배울 때 가장 기본이 되는 “Hello World”를 콘솔에 출력하는 프로그램으로 리액티브 스트림즈의 구현체인 Reactor를 통해 “Hello, Reactive”를 출력함
- Puhlisher의 역할을 하는 것이 Mono
    - Publisher는 데이터를 emit하는 역할을 하며, Subsciber는 Publisher가 emit한 데이터를 전달받아서 소비하는 역할을 함
- Subscriber의 역할을 하는 것은 subscribe() 메서드 내부에 정의된 람다 표현식인 message -> System.out.println(message)
- Java의 Stream에서 메서드 체인 형태로 사용할 수 있는 것처럼 리액티브 프로그래밍 역시 메서드 체인을 구성할 수 있음

<br>

```java
import reactor.core.publisher.Mono;

// 리액티브 프로그래밍 기본 구조
public class HelloReactiveExample02 {
    public static void main(String[] args) {
        Mono
            .just("Hello, Reactive")
            .subscribe(message -> System.out.println(message));
    }
}
```

<br>

- HelloReactiveExample01 클래스의 코드를 하나의 메서드 체인 형태로 표현한 코드

<br>

### 리액티브 프로그래밍에서 사용되는 용어 정의

```java
import reactor.core.publisher.Flux;

import java.util.List;

public class ReactiveGlossaryExample {
    public static void main(String[] args) {
        Flux
            .fromIterable(List.of(1, 3, 6, 7, 8, 11))
            .filter(number -> number > 4 && (number % 2 == 0))
            .reduce((n1, n2) -> n1 + n2)
            .subscribe(System.out::println);

    }
}
```

<br>

- Publisher
    - 리액티브 스트림즈 사양에서도 확인한 것처럼 Publisher는 데이터를 내보내는 주체를 의미함
    - Flux가 Publisher
- Emit
    - Publisher가 데이터를 내보내는 것을 Emit이라 함
- Subscriber
    - Subscriber는 Publisher가 emit한 데이터를 전달받아서 소비하는 주체를 의미함
    - subscribe(System.out::println) 중에서 System.out::println이 Subscriber에 해당됨
    - 람다 표현식을 메서드 레퍼런스로 축약하지 않았다면 람다 표현식 자체가 Subscriber에 해당됨
- Subscribe
    - Subscribe는 구독을 의미함
    - subscribe() 메서드를 호출하면 구독을 하는 것
- Signal
    - Signal은 Publisher가 발생 시키는 이벤트를 의미함
    - subscribe() 메서드가 호출되면 Publisher인 Flux는 숫자 데이터를 하나씩 하나씩 emit하는데, 이때 숫자 데이터를 하나씩 emit하는 자체를 리액티브 프로그래밍에서는 이벤트가 발생하는 것으로 간주하며, 이 이벤트 발생을 다른 컴포넌트에게 전달하는 것을 Signal을 전송한다라고 표현함
- Operator
    - Operator는 리액티브 프로그래밍에서 어떤 동작을 수행하는 메서드를 의미함
    - fromIterable(), filter(), reduce() 등 메서드 하나하나를 Operator라고 함
    - 우리말로 연산자라는 표현을 사용함
- Sequence
    - Sequence는 Operator 체인으로 표현되는 데이터의 흐름을 의미함
    - Operator 체인으로 작성된 코드 자체를 하나의 Sequence라고 함
- Upstream / Downstream
    - Sequence 상의 특정 Operator를 기준으로 위쪽의 Sequence 일부를 Upstream이라고 하며, 아래쪽 Sequence 일부를 Downstream이라고 표현함
    - filter() Operator를 기준에서 보면 filter() Operator 위 쪽의 fromIterable()은 Upstream이 됨
    - filter() Operator 아래쪽의 reduce() Operator는 Downstream이 됨