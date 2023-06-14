태그: #Spring 
연결문서: [Spring WebFlux - Spring WebFlux 1](Spring%20WebFlux%20-%20Spring%20WebFlux%201.md)

# Project Reactor

## Operators
- Operator는 Reactor에서 가장 중요한 구성요소
- 리액티브 스트림즈의 구현체인 Reactor 역시 다양한 종류의 Operator를 지원함
- Operator의 종류가 너무 많기 때문에 적절한 상황에 맞게 사용할 수 있도록 Operator가 상황별로 분류가 되어 있음

### 상황별로 분류된 Operator 목록 리뷰
- 새로운 Sequence를 생성(Creating)하고자 할 경우
    - just()
    - fromStream()
    - fromIterable()
    - fromArray()
    - range()
    - interval()
    - empty()
    - never()
    - defer()
    - using()
    - generate()
    - create()
<br>

- 기존 Sequence에서 변환 작업(Transforming)이 필요한 경우
    - map()
    - flatMap()
    - concat()
    - collectList()
    - collectMap()
    - merge()
    - zip()
    - then()
    - switchIfEmpty()
    - and()
    - when()
<br>

- Sequence 내부의 동작을 확인(Peeking)하고자 할 경우
    - doOnSubscribe
    - doOnNext()
    - doOnError()
    - doOnCancel()
    - doFirst()
    - doOnRequest()
    - doOnTerminate()
    - doAfterTerminate()
    - doOnEach()
    - doFinally()
    - log()
<br>

- Sequence에서 데이터 필터링(Filtering)이 필요한 경우
    - filter()
    - ignoreElements()
    - distinct()
    - take()
    - next()
    - skip()
    - sample()
    - single()
<br>

- 에러를 처리(Handling errors)하고자 할 경우
    - error()
    - timeout()
    - onErrorReturn()
    - onErrorResume()
    - onErrorMap()
    - doFinally()
    - retry()

<br>

#### Operator 용 샘플 데이터

```java
import java.util.Arrays;
import java.util.List;

public class SampleData {
    public static List<Coffee> coffeeList = List.of(
            new Coffee("아메리카노", "Americano", 2500, "AMR"),
            new Coffee("카페라떼", "CafeLatte", 3500, "CFR"),
            new Coffee("바닐라 라떼", "Vanilla Latte", 4500, "VNL"),
            new Coffee("카라멜 마끼아또", "Caramel Macchiato", 5500, "CRM"),
            new Coffee("에스프레소", "Espresso", 5000, "ESP")
    );

    // A 지점 카페의 월별 매출
    public static final List<Integer> salesOfCafeA = Arrays.asList(
            5_500_000, 4_200_000, 3_500_000, 5_000_000, 3_700_000, 4_000_000, 5_300_000, 5_800_000,
            3_500_000, 2_900_000, 5_400_000, 4_900_000
    );

    // B 지점 카페의 월별 매출
    public static final List<Integer> salesOfCafeB = Arrays.asList(
            2_500_000, 3_100_000, 4_300_000, 3_500_000, 3_200_000, 2_800_000, 3_100_000, 4_200_000,
            3_100_000, 3_200_000, 3_400_000, 4_100_000
    );

    // C 지점 카페의 월별 매출
    public static final List<Integer> salesOfCafeC = Arrays.asList(
            5_500_000, 5_100_000, 5_300_000, 5_500_000, 4_700_000, 4_800_000, 4_100_000, 5_200_000,
            5_100_000, 4_200_000, 4_400_000, 5_100_000
    );
}
```

<br>

#### 새로운 Sequence를 생성(Creating)하고자 할 경우
- fromStream
    - Java의 Stream을 입력으로 전달받아 emit하는 Operator
    - (1)에서 fromStream()의 입력으로 Stream을 전달히며, 전달받은 Stream이 포함하고 있는 데이터를 차례대로 emit함
    - reduce() Operator는 Upstream에서 emit 된 두 개의 데이터를 순차적으로 누적 처리할 수 있는 Operator로, (2)에서는 Upstream으로부터 전달받은 두 개의 숫자를 합하는 과정을 반복해서 총합계를 구하고 있음
<br>

```java
import reactor.core.publisher.Flux;

import java.util.stream.Stream;

/**
 * fromStream() 기본 예제
 */
public class FromStreamExample01 {
    public static void main(String[] args) {
        Flux
            .fromStream(Stream.of(200, 300, 400, 500, 600))  // (1)
            .reduce((a, b) -> a + b)                         // (2)
            .subscribe(System.out::println);
    }
}

// 실행 결과
2000
```

<br>

- fromIterable()
    - Java의 Iterable을 입력으로 전달받아 emit하는 Operator
    - List, Map, Set 등의 컬렉션을 fromIterable()의 파라미터로 전달할 수 있음
    - SampleData.coffeeList를 통해 Coffee 클래스의 객체를 원소로 가지는 List를 fromIterable() Operator의 입력으로 전달받은 후, coffee 한글명과 가격을 차례대로 출력함
<br>

```java
import com.operators.sample_data.SampleData;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

import java.util.List;
import java.util.stream.Stream;

/**
 * fromIterable() 기본 예제
 */
@Slf4j
public class FromIterableExample01 {
    public static void main(String[] args) {
        Flux
            .fromIterable(SampleData.coffeeList)
            .subscribe(coffee -> log.info("{} : {}", coffee.getKorName(), coffee.getPrice()));
    }
}

// 실행 결과
[main] INFO com.operators.create.FromIterableExample01 - 아메리카노 : 2500
[main] INFO com.operators.create.FromIterableExample01 - 카페라떼 : 3500
[main] INFO com.operators.create.FromIterableExample01 - 바닐라 라떼 : 4500
[main] INFO com.operators.create.FromIterableExample01 - 카라멜 마끼아또 : 5500
[main] INFO com.operators.create.FromIterableExample01 - 에스프레소 : 5000
```

<br>

- create()
    - 프로그래밍 방식으로 Signal 이벤트를 발생시키는 Operator
    - Signal이라는 용어는 리액티브 프로그래밍 용어 정의에서 설명한 것처럼 Publisher가 발생시키는 이벤트를 의미함
    - 일반적으로 Publisher가 데이터를 emit할 경우, onNext Signal 이벤트를 전송한다라고 표현함
    - create() Operator는 한 번에 여러 건의 데이터를 비동기적으로 emit할 수 있는 Operator
    - create() Operator의 파라미터는 FluxSink라는 람다 파라미터를 가지는 람다 표현식
        - (1)의 FluxSink는 Flux나 Mono에서 just(), fromIterable() 같은 데이터 생성 Operator에 데이터소스를 전달하면 내부에서 알아서 데이터를 emit 하는 등의 Sequence를 진행하는 것이 아니라 프로그래밍 방식으로(programmatically) 직접 Signal 이벤트를 발생시켜서 Sequence를 진행하도록 해주는 기능을 함
        - create() Operator 내부에서 FluxSink의 객체를 통해 모든 작업을 진행함
    - (2)의 onRequest()는 Subscriber에서 데이터를 요청하면 onRequest()의 파라미터인 람다 표현식이 실행됨
        - (3)에서 for문을 순회하면서 next() 메서드로 List source의 원소를 emit함
        - (4)에서는 List source 원소를 모두 emit했으므로, Sequence를 종료하기 위해 complete()를 호출함
        - (5)의 onDispose()는 Sequence가 완전히 종료되기 직전에 호출되며, sequence 종료 직전 후처리 작업을 할 수 있음
<br>

```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.publisher.FluxSink;

import java.util.Arrays;
import java.util.List;

/**
 * create() Operator 기본 예제
 */
@Slf4j
public class CreateExample {
    private static List<Integer> source= Arrays.asList(1, 3, 5, 7, 9, 11, 13, 15, 17, 19);

    public static void main(String[] args) {
        Flux.create((FluxSink<Integer> sink) -> {   // (1)
            // (2)
            sink.onRequest(n -> {
                for (int i = 0; i < source.size(); i++) {
                    sink.next(source.get(i));   // (3)
                }
                sink.complete();    // (4)
            });

            // (5)
            sink.onDispose(() -> log.info("# clean up"));
        }).subscribe(data -> log.info("# onNext: {}", data));
    }
}

// 실행 결과
[main] INFO com.operators.create.CreateExample - # onNext: 1
[main] INFO com.operators.create.CreateExample - # onNext: 3
[main] INFO com.operators.create.CreateExample - # onNext: 5
[main] INFO com.operators.create.CreateExample - # onNext: 7
[main] INFO com.operators.create.CreateExample - # onNext: 9
[main] INFO com.operators.create.CreateExample - # onNext: 11
[main] INFO com.operators.create.CreateExample - # onNext: 13
[main] INFO com.operators.create.CreateExample - # onNext: 15
[main] INFO com.operators.create.CreateExample - # onNext: 17
[main] INFO com.operators.create.CreateExample - # onNext: 19
[main] INFO com.operators.create.CreateExample - # clean up
```

<br>

#### 기존 Sequence에서 변환 작업(Transforming)이 필요한 경우
- flatMap()
    - flatMap() 내부로 들어오는 데이터 한 건당 하나의 Sequence가 생성됨
    - Upstream에서 2개의 데이터를 emit하고, flatMap() 내부에서 3개의 데이터를 emit하는 Sequence가 있다면 Downstream으로 emit되는 데이터는 총 6개(2 x 3)가 됨
    - flatMap() 내부에서 정의하는 Sequence를 Inner Sequece라고 부름
    - (1)에서 range() Operator를 이용해서 구구단 2단부터 7단까지 출력하도록 지정함
    - (2)와 같이 flatMap() 내부에서 range() Operator로 하나의 단을 출력하도록 1부터 9까지 숫자를 지정함
    - (3)에서 flatMap() 내부의 Inner Sequence를 처리할 스레드를 할당하며, 전체 Sequence는 여러 개의 스레드가 비동기적으로 동작함
    - (4)에서는 구구단 형식으로 문자열을 구성함
    - flatMap() Operator 예제 코드 실행 결과를 보면 2단부터 7단까지 구구단이 출력됨
        - 2단부터 7단까지 차례대로 출력되는 것이 아니라 실행 결과가 뒤섞여서 출력됨
        - flatMap() Operator에서 추가 스레드를 할당할 경우, 작업의 처리 순서를 보장하지 않음
<br>

```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;

/**
 * flatMap() 기본 예제
 */
@Slf4j
public class FlatMapExample01 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .range(2, 6)         // (1)
            .flatMap(dan -> Flux
                    .range(1, 9)  // (2)
                    .publishOn(Schedulers.parallel())   // (3)
                    .map(num -> dan + " x " + num + " = " + dan * num)) // (4)
            .subscribe(log::info);

        Thread.sleep(100L);
    }
}

// 실행 결과
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 1 = 7
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 2 = 14
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 3 = 21
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 4 = 28
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 5 = 35
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 6 = 42
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 7 = 49
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 8 = 56
[parallel-6] INFO com.operators.transformation.FlatMapExample01 - 7 x 9 = 63
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 1 = 2
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 1 = 3
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 2 = 6
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 3 = 9
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 4 = 12
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 5 = 15
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 6 = 18
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 7 = 21
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 8 = 24
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 3 x 9 = 27
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 1 = 4
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 2 = 8
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 3 = 12
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 4 = 16
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 5 = 20
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 6 = 24
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 7 = 28
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 8 = 32
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 4 x 9 = 36
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 1 = 5
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 2 = 10
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 3 = 15
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 4 = 20
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 5 = 25
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 6 = 30
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 7 = 35
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 8 = 40
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 5 x 9 = 45
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 1 = 6
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 2 = 12
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 3 = 18
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 4 = 24
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 5 = 30
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 6 = 36
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 7 = 42
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 8 = 48
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 6 x 9 = 54
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 2 = 4
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 3 = 6
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 4 = 8
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 5 = 10
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 6 = 12
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 7 = 14
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 8 = 16
[parallel-1] INFO com.operators.transformation.FlatMapExample01 - 2 x 9 = 18
```

<br>

- concat()
    - 입력으로 전달하는 Publisher의 Sequence를 연결해서 차례대로 데이터를 emit함
    - Reactor의 concat() Operator는 논리적으로 하나의 Sequence로 이어 붙인다는 의미이기 때문에 이어 붙인 Sequecne에서 시간 순서대로 데이터를 차례대로 emit함
    - 기본 예제 1에서는 concat() Operator의 입력으로 두 개의 Flux Sequence를 전달했기 때문에 두 개의 Sequence를 이어 붙여서 논리적으로 하나의 Sequence로 동작하게 됨
    - 기본 예제 2는 concat() Operator를 이용해서 3개 카페 지점의 월별 매출액을 모두 하나의 Sequence로 연결 한 다음 카페의 전체 매출액을 계산하는 예제로, 3개 카페의 월별 매출액은 reduce() Operator로 누적해서 전체 매출액을 계산함
        - reactor-core 모듈에는 수학 계산을 위한 Operator는 존재하지 않기 때문에 reduce() Operator를 사용해서 카페 지점의 월별 매출 합계를 계산함
        - 하지만 reactor-extra 모듈에는 수학 계산과 관련된 작업을 처리할 수 있는 MathFlux라는 Reactor 타입을 포함하고 있으며, MathFlux의 Operator를 사용하면 숫자 데이터의 합계나 평균 등을 손쉽게 구할 수 있음
<br>

```java
import reactor.core.publisher.Flux;

/**
 * concat() 기본 예제 1
 */
public class ConcatExample01 {
    public static void main(String[] args) {
        Flux
            .concat(Flux.just("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"),
                    Flux.just("Saturday", "Sunday"))
            .subscribe(System.out::println);
    }
}

// 실행 결과
Monday
Tuesday
Wednesday
Thursday
Friday
Saturday
Sunday
```

<br>

```java
import com.codestates.example.operators.sample_data.SampleData;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

/**
 * concat() 기본 예제 2
 */
@Slf4j
public class ConcatExample02 {
    public static void main(String[] args) {
        Flux
            .concat(Flux.fromIterable(SampleData.salesOfCafeA),
                    Flux.fromIterable(SampleData.salesOfCafeB),
                    Flux.fromIterable(SampleData.salesOfCafeC))
                .reduce((a, b) -> a + b)
            .subscribe(data -> log.info("# total sales: {}", data));
    }
}

// 실행 결과
[main] INFO com.operators.transformation.ConcatExample02 - # total sales: 153200000
```

<br>

- zip()
    - 입력으로 전달되는 여러 개의 Publisher Sequence에서 emit 된 데이터를 결합하는 Operator
    - 결합은 각 Publisher가 emit하는 데이터를 하나씩 전달받아서 새로운 데이터를 만든 후에 Downstream으로 전달한다는 의미
    - 각각의 Sequence에서 emit되는 데이터 중에서 같은 차례(index)의 데이터들이 결합됨
    - 각 Sequence에서 emit되는 데이터의 시점이 다르기 때문에 결합되어야 하는 데이터(같은 index)가 emit이 될 때까지 기다렸다가 결합함
    - (1)과 (2)에서 사용한 interval() Operator는 파라미터로 전달한 시간(Duration.ofMillis(…))을 주기로 해서 0부터 1씩 증가한 숫자를 emit하는 Operator
        - interval() Operator가 끊임없이 숫자를 emit하는 Operator이기 때문에 take() Operator를 이용해서 take() Operator에 파라미터로 지정한 숫자만큼의 데이터만 emit하고 종료하도록 했음
        - (1)의 Sequence는 0.2초에 한 번씩 데이터를 emit하고, (2)의 Sequence는 0.4초에 한 번씩 데이터를 emit하기 때문에 (1)의 Sequence가 매 번 조금 더 빨리 데이터를 emit함
        - (1), (2)의 Sequence는 (3)에서 zip() Operator의 입력으로 전달되기 때문에 두 Sequence의 emit 시점이 매번 다르더라도 emit 시점이 늦은 데이터가 emit될 때까지 대기했다가 두 개의 데이터를 전달받게 됨
    - 실행 결과를 보면 총 네 개의 데이터가 Subscriber에게 전달되며, (2)의 Sequence가 총 6개의 데이터를 emit하지만 (1)의 Sequence가 4개의 데이터만 emit하고 종료되기 때문에 (2)의 Sequence는 결합할 대상이 없으므로 남은 두 개 데이터는 폐기됨
<br>

```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

import java.time.Duration;

/**
 * zip() 기본 예제
 */
@Slf4j
public class ZipExample01 {
    public static void main(String[] args) throws InterruptedException {
        // (1)
        Flux<Long> source1 = Flux.interval(Duration.ofMillis(200L)).take(4);

        // (2)
        Flux<Long> source2 = Flux.interval(Duration.ofMillis(400L)).take(6);

        Flux
            .zip(source1, source2, (data1, data2) -> data1 + data2)   // (3)
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(3000L);
    }
}

// 실행 결과
[parallel-2] INFO com.operators.transformation.ZipExample01 - # onNext: 0
[parallel-2] INFO com.operators.transformation.ZipExample01 - # onNext: 2
[parallel-2] INFO com.operators.transformation.ZipExample01 - # onNext: 4
[parallel-2] INFO com.operators.transformation.ZipExample01 - # onNext: 6
```

<br>

#### Sequence 내부의 동작을 확인(Peeking)하고자 할 경우
- doOnNext()
    - 데이터 emit 시 트리거 되어 부수 효과(side-effect)를 추가할 수 있는 Operator
    - 실제 emit 된 데이터를 가지고 무언가 처리 작업을 하지만 Downstream으로 전달되는 것은 emit 된 데이터이지 `doOnNext()` Operator에서 처리된 작업의 결과 값은 아님
    - 함수형 프로그래밍 세계에서 부수 효과(side-effect)는 어떤 동작을 실행하되 리턴 값이 없는 것을 의미하며, doOnNext()는 리턴 값이 없음
    - doOnNext()는 주로 로깅(로그를 기록 또는 출력하는 작업)에 사용되지만 데이터를 emit하면서 필요한 추가 작업이 있다면 doOnNext()에서 처리할 수 있음
    - 아래는 doOnNext()를 이용해서 emit되는 데이터의 유효성 검증을 진행하는 예제 코드
    - Spring MVC에서의 DTO 클래스에 대한 유효성 검증과 유사하게 Reactor Sequence 상에서 emit 된 데이터의 유효성 검증을 진행하고자 한다면 (1)과 같이 doOnNext() Operator에서 수행할 수 있음
<br>

```java
import com.operators.sample_data.Coffee;
import com.operators.sample_data.SampleData;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

/**
 * doOnNext() 기본 예제
 */
@Slf4j
public class DoOnNextExample01 {
    public static void main(String[] args) {
        Flux
                .fromIterable(SampleData.coffeeList)
                .doOnNext(coffee -> validateCoffee(coffee))    // (1)
                .subscribe(data -> log.info("{} : {}", data.getKorName(), data.getPrice()));
    }

    private static void validateCoffee(Coffee coffee) {
        if (coffee == null) {
            throw new RuntimeException("Not found coffee");
        }
        // TODO 유효성 검증에 필요한 로직을 필요한 만큼 추가할 수 있음
    }
}

// 실행 결과
[main] INFO com.operators.peeking.DoOnNextExample01 - 아메리카노 : 2500
[main] INFO com.operators.peeking.DoOnNextExample01 - 카페라떼 : 3500
[main] INFO com.operators.peeking.DoOnNextExample01 - 바닐라 라떼 : 4500
[main] INFO com.operators.peeking.DoOnNextExample01 - 카라멜 마끼아또 : 5500
[main] INFO com.operators.peeking.DoOnNextExample01 - 에스프레소 : 5000
```

<br>

- log()
    - log() Operator는 Publisher에서 발생하는 Signal 이벤트를 로그로 출력해 주는 역할을 함
    - log() Operator는 원하는 만큼 추가할 수 있음
    - 실행 결과를 보면 Publisher에서 발생한 Signal 이벤트에 대한 내용이 로그로 출력된 것을 확인할 수 있음
        - 구독 시점에 onSubscribe Signal 이벤트가 발생하며, 실행 결과에서는 onSubscribe Signal 이벤트가 두 번 발생
        - 데이터 요청 시, request Signal 이벤트가 발생하며, 실행 결과에서는 request 이벤트가 두 번 발생함
        - Publisher가 데이터를 emit할 때 onNext Signal 이벤트가 발생하며, 실행 결과에서는 총 다섯 개의 데이터가 emit 되었으므로 다섯 번의 onNext Signal 이벤트가 발생함
        - Publisher의 데이터 emit이 정상적으로 종료되면 onComplete Signal 이벤트가 발생하며, 실행 결과에서는 총 두 번의 onComplete Signal 이벤트가 발생함
<br>

```java
import reactor.core.publisher.Flux;

import java.util.stream.Stream;

public class LogExample01 {
    public static void main(String[] args) {
        Flux
            .fromStream(Stream.of(200, 300, 400, 500, 600))
            .log()
            .reduce((a, b) -> a + b)
            .log()
            .subscribe(System.out::println);
    }
}

// 실행 결과
[main] INFO reactor.Flux.Stream.1 - | onSubscribe([Synchronous Fuseable] FluxIterable.IterableSubscription)
[main] INFO reactor.Mono.Reduce.2 - | onSubscribe([Fuseable] MonoReduce.ReduceSubscriber)
[main] INFO reactor.Mono.Reduce.2 - | request(unbounded)
[main] INFO reactor.Flux.Stream.1 - | request(unbounded)
[main] INFO reactor.Flux.Stream.1 - | onNext(200)
[main] INFO reactor.Flux.Stream.1 - | onNext(300)
[main] INFO reactor.Flux.Stream.1 - | onNext(400)
[main] INFO reactor.Flux.Stream.1 - | onNext(500)
[main] INFO reactor.Flux.Stream.1 - | onNext(600)
[main] INFO reactor.Flux.Stream.1 - | onComplete()
[main] INFO reactor.Mono.Reduce.2 - | onNext(2000)
2000
[main] INFO reactor.Mono.Reduce.2 - | onComplete()
```

<br>

#### 에러를 처리(Handling errors)하고자 할 경우
- error()
    - 의도적으로 onError Signal 이벤트를 발생시킬 때 사용할 수 있는 Operator
    - Reactor Sequence 상에서 의도적으로 예외를 던져서 onError Signal 이벤트를 발생시키는 데 사용할 수 있음
    - (1)의 justOrEmpty() Operator는 파라미터로 전달되는 데이터소스가 null 이어도 에러가 발생하지 않으며, just() Operator는 null 데이터를 emit하면 에러가 발생함
    - (2)의 switchIfEmpty() Operator는 Upstream에서 전달되는 데이터가 null이면 대체 동작을 수행할 수 있음
        - 유효하지 않은 커피 객체(예를 들어 null)가 전달되면 error() Operator를 사용해서 onError Signal 이벤트를 발생하도록 함
        - findVerifiedCoffee() 메서드가 null을 리턴하기 때문에 (2)에서 onError Signal 이벤트가 전송되고, (3)에서는 error 객체를 전달받아서 에러 메시지를 출력하고 있음
<br>

```java
import com.operators.sample_data.Coffee;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Mono;

/**
 * 에러 기본 예제
 */
@Slf4j
public class ErrorExample01 {
    public static void main(String[] args) {
        Mono.justOrEmpty(findVerifiedCoffee())  // (1)
                .switchIfEmpty(Mono.error(new RuntimeException("Not found coffee")))  // (2)
                .subscribe(
                        data -> log.info("{} : {}", data.getKorName(), data.getPrice()),
                        error -> log.error("# onError: {}", error.getMessage()));  // (3)
    }

    private static Coffee findVerifiedCoffee() {
        // TODO 데이터베이스에서 Coffee 정보를 조회할 수 있음

        return null;
    }
}

// 실행 결과
[main] ERROR com.operators.errors.ErrorExample01 - # onError: Not found coffee
```

<br>

- timeout(), retry()
    - timeout() Operator는 입력으로 주어진 시간 동안 emit되는 데이터가 없으면 onError Signal 이벤트를 발생시킴
    - retry() Operator는 Sequence 상에서 에러가 발생할 경우, 입력으로 주어진 숫자만큼 재구독해서 Sequence를 다시 시작함
    - timeout()과 retry() Operator를 이용해서 일정 시간 내에 데이터가 emit 되지 않으면 다시 시도하는 예제
        - (1)의 delayElements() Operator는 입력으로 주어진 시간만큼 각각의 데이터 emit을 지연시키는 Operator로, Coffee 객체는 0.5초에 한 번씩 emit됨
        - (2)에서 emit되는 세 번째 커피 정보(coffee) 의도적으로 2초 더 지연시킴
        - (3)에서는 2초 안에 데이터가 emit되지 않으면 onError Signal 이벤트가 발생하도록 지정했기 때문에 (2)에서 총 2.5초가 지연되어 세 번째 커피 정보(coffee)는 Downstream으로 emit되지 않음
        - onError Signal 이벤트가 발생했기 때문에 모든 Sequence가 종료되어야 하지만 (4)에서 retry() Operator를 추가했기 때문에 1회 재구독을 해서 Sequence를 다시 시작함
        - 이제 timeout이 발생할 이유가 없으므로 데이터는 정상적으로 emit 됨
        - (5)에서 emit 된 데이터를 Set&#60;Coffee&#62;으로 변환하는 이유는 timeout이 되기 전에 이미 emit 된 데이터가 있으므로 재구독 후, 다시 emit 된 데이터에 동일한 데이터가 있으므로 중복을 제거하기 위함
        - 실행 결과를 보면, 세 번째 데이터가 timeout이 되어 Drop이 된 것을 확인할 수 있음
        - retry()를 통해 재구독이 처리가 되어 Sequence가 다시 시작되고, Subscriber는 최종적으로 중복이 제거된 커피 정보를 전달받는 것을 확인할 수 있음
<br>

```java
import com.operators.sample_data.Coffee;
import com.operators.sample_data.SampleData;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

import java.time.Duration;
import java.util.stream.Collectors;

/**
 * timeout(), retry() 기본 예제
 */
@Slf4j
public class TimeoutRetryExample01 {
    public static void main(String[] args) throws InterruptedException {
        getCoffees()
                .collect(Collectors.toSet())   // (5)
                .subscribe(bookSet -> bookSet
                                        .stream()
                                        .forEach(data ->
                                                log.info("{} : {}", data.getKorName(), data.getPrice())));

        Thread.sleep(12000);
    }

    private static Flux<Coffee> getCoffees() {
        final int[] count = {0};
        return Flux
                .fromIterable(SampleData.coffeeList)
                .delayElements(Duration.ofMillis(500)) // (1)
                .map(coffee -> {
                    try {
                        count[0]++;
                        if (count[0] == 3) {     // (2)
                            Thread.sleep(2000);
                        }
                    } catch (InterruptedException e) {
                    }

                    return coffee;
                })
                .timeout(Duration.ofSeconds(2))   // (3)
                .retry(1)     // (4)
                .doOnNext(coffee -> log.info("# getCoffees > doOnNext: {}, {}",
                        coffee.getKorName(), coffee.getPrice()));
    }
}

// 실행 결과
[main] DEBUG reactor.util.Loggers - Using Slf4j logging framework
[parallel-2] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 아메리카노, 2500
[parallel-4] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 카페라떼, 3500
[parallel-6] DEBUG reactor.core.publisher.Operators - onNextDropped: com.operators.sample_data.Coffee@76fb8d6f
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 아메리카노, 2500
[parallel-2] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 카페라떼, 3500
[parallel-4] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 바닐라 라떼, 4500
[parallel-6] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 카라멜 마끼아또, 5500
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - # getCoffees > doOnNext: 에스프레소, 5000
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - 아메리카노 : 2500
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - 바닐라 라떼 : 4500
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - 카페라떼 : 3500
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - 카라멜 마끼아또 : 5500
[parallel-8] INFO com.operators.errors.TimeoutRetryExample01 - 에스프레소 : 5000
```