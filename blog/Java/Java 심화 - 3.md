태그: #Java 
연결문서: [Java 심화 - 4](Java%20심화%20-%204.md)

# Java 심화

## 스트림(Stream)

> 배열, 컬렉션의 저장 요소를 하나씩 참조해서 람다식으로 처리할 수 있도록 해주는 반복자이다.
> 
> 스트림을 사용하면 List, Set, Map, 배열 등 다양한 데이터 소스로부터 스트림을 만들 수 있고, 이를 표준화된 방법으로 다룰 수 있다.
> 
> 스트림은 데이터 소스를 다루는 풍부한 메소드를 제공하고 이를 활용하면 다량의 데이터에 복잡한 연산을 수행하면서 가독성과 재사용성이 높은 코드를 작성할 수 있다.

### 스트림의 특징

1.  스트림 처리 과정은 생성, 중간 연산, 최종 연산 세 단계의 파이프라인으로 구성
2.  스트림은 원본 데이터 소스를 변경하지 않음(read-only)
3.  일회용(onetime-only)
4.  내부 반복자

### 스트림의 생성

> 스트림으로 데이터를 처리하기 위해서는 가정 먼저 스트림을 생성해야 한다.
> 
> 스트림을 생성할 수 있는 데이터 소스는 배열, 컬렉션, 임의의 수, 특정 범위의 정수 등으로 다양하며 이에 따라 스트림의 생성 방법에 조금씩 차이가 있다.

#### 배열 스트림 생성

-   **Arrays.stream()**
    -   Arrays 클래스에는 int, long, double과 같은 기본형 배열을 데이터 소스로 스트림을 생성하는 메소드도 있음

```
public class StreamCreator {

       public static void main(String[] args) {
           // 문자열 배열 선언 및 할당
           String[] arr = new String[]{"Mason", "Serena"};

           // 문자열 스트림 생성
           Stream<String> stream = Arrays.stream(arr);

           // 출력
           stream.forEach(System.out::println);

       }
   }

   // 출력값
   Mason
   Serena
```

-   **Stream.of()**

```
import java.util.stream.Stream;

public class StreamCreator {

    public static void main(String[] args) {
        // 문자열 배열 선언 및 할당
        String[] arr = new String[]{"Mason", "Serena"};

        // 문자열 스트림 생성
        Stream<String> stream = Stream.of(arr);

        // 출력
        stream.forEach(System.out::println);

    }
}

// 출력값
Mason
Serena
```

#### 컬렉션 스트림 생성

-   Collection으로부터 확장된 하위 클래스인 List와 Set을 구현한 컬렉션 클래스들은 모두 stream() 메서드를 사용하여 스트림을 생성할 수 있음

```
// List 타입의 스트램 생성 과정
import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

public class StreamCreator {

    public static void main(String[] args) {
                // 요소들을 리스트
        List<Integer> list = Arrays.asList(1, 3, 5, 7, 9);
        Stream<Integer> stream = list.stream();

        stream.forEach(System.out::print);
    }
}

//출력값
13579
```

#### 임의의 수 스트림 생성

-   난수를 생성하는 자바의 기본 내장 클래스 Random 클래스 안에는 해당 타입의 난수들을 반환하는 스트림을 생성하는 메서드들이 정의되어 있음
    -   int() 메서드의 경우 int형의 범위 안에 있는 난수들을 무한대로 생성하여 IntStream 타입의 스트림으로 반환

```
// 무한 스트림(infinite stream)
// 스트림의 크기가 정해지지 않은 것
import java.util.Random;
import java.util.stream.IntStream;

public class StreamCreator {

    public static void main(String[] args) {
        // 난수 생성
        IntStream ints = new Random().ints();
        ints.forEach(System.out::println);
    }
}
```

```
// 스트림의 범위 제한 예시
// limit() 메서드 또는 매개변수로 스트림의 사이즈를 전달
import java.util.Random;
import java.util.stream.IntStream;

public class StreamCreator {

    public static void main(String[] args) {

            // 스트림 생성의 범위를 10개로 제한
        IntStream ints = new Random().ints(10);
                IntStream ints = new Random().ints().limit(10); 
        ints.forEach(System.out::println);
    }
}
```

```
// 특정 범위의 정수 값을 스트림으로 생성해서 반환
import java.util.stream.IntStream;

public class StreamCreator {

    public static void main(String[] args) {

        //특정 범위의 정수
        IntStream intStream = IntStream.rangeClosed(1, 5);
        intStream.forEach(System.out::println);
    }
}

//출력값
12345
```

### 스트림의 중간 연산

> 스트림 생성 이후, 스트림 내의 요소를 원하는 형태에 알맞게 가공한다.
> 
> 필터링, 매핑, 정렬 등이 가장 빈번하게 사용되는 중간 연산자이다.

#### 필터링( filter(), distinct() )

> 필요에 따라 조건에 맞는 데이터들만을 정제하는 중간 연산자이다.

-   **distinct()**
    -   Stream의 요소들에 중복된 데이터가 존재하는 경우, 중복을 제거하기 위해 사용
-   **filter()**
    -   Stream에서 조건에 맞는 데이터만을 정제하여 더 작은 컬렉션을 생성
    -   filter() 메서드에는 매개값으로 조건이 주어지고, 조건이 참이 되는 요소만 필터링하며 조건은 람다식을 사용하여 정의할 수 있음

```
import java.util.Arrays;
import java.util.List;

public class FilteringExample {
    public static void main(String[] args) throws Exception {

        List<String> names = Arrays.asList("김일", "이이", "박삼", "김일", "이오");

        names.stream()
                .distinct() //중복 제거
                .forEach(element -> System.out.println(element));
        System.out.println();

        names.stream()
                .filter(element -> element.startsWith("김")) // 김씨 성을 가진 요소만 필터링 
                .forEach(element -> System.out.println(element));
        System.out.println();

        names.stream()
                .distinct() //중복제거
                .filter(element -> element.startsWith("김")) // 김씨 성을 가진 요소만 필터링 
                .forEach(element -> System.out.println(element));
    }
}

// 출력값
김일
이이
박삼
이오

김일
김일

김일
```

#### 매핑( map() )

> 스트림 내 요소들에서 원하는 필드만 추출하거나 특정 형태로 변환할 때 사용하는 중간 연산자이다.

-   **map()**

```
import java.util.Arrays;
import java.util.List;

public class IntermediateOperationExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("mason", "serena", "hana");
        names.stream()
                .map(element -> element.toUpperCase()) // 요소들을 하나씩 대문자로 변환
                .forEach(element->System.out.println(element));
    }
}

// 출력값
MASON
SERENA
HANA
```

```
import java.util.Arrays;
import java.util.List;

public class IntermediateOperationExample {
    public static void main(String[] args)
    {
        List<Integer> list = Arrays.asList(1, 3, 5, 7);

        // 각 요소에 5을 곱한 값을 반환
        list.stream().map(number -> number * 5).forEach(System.out::println);
    }

}

// 출력값
5
15
25
35
```

```
// 주어진 이중 배열
String[][] namesArray = new String[][]{{"mason", "serena"}, {"hana", "nana"}};

// 기대하는 출력값
mason
serena
hana
nana

// map() 사용
// 중첩 스트림(Stream<Stream<String>>)을 반환하여 스트램 객체의 값을 반환
        Arrays.stream(namesArray)
                .map(inner -> Arrays.stream(inner))
                .forEach(System.out::println);

// 출력값
java.util.stream.ReferencePipeline$Head@3cb5cdba
java.util.stream.ReferencePipeline$Head@56cbfb61

// map() 사용 수정
// Stream<String>을 반환
        Arrays.stream(namesArray)
                .map(inner -> Arrays.stream(inner))
                .forEach(names -> names.forEach(System.out::println));

// 출력값
mason
serena
hana
nana
```

-   **flatMap()**
    -   중첩 구조를 제거하고 단일 컬렉션(Stream<String>)을 만들어주는 역할

```
// 주어진 이중 배열
String[][] namesArray = new String[][]{{"mason", "serena"}, {"hana", "nana"}};

Arrays.stream(namesArray).flatMap(Arrays::stream).forEach(System.out::println);

// 출력값
mason
serena
hana
nana
```

#### 정렬( sorted() )

> 정렬을 할 때 사용하는 중간 연산자이다.
> 
> sorted() 메서드를 사용하여 정렬을 할 떄에는 괄호 안에 Comparator 라는 인터페이스에 정의된 static 메서드와 디폴트 메서드를 사용하여 간편하게 정렬 작업을 수행할 수 있다.
> 
> 고할호 안에 아무 값도 넣지 않은 상태로 호출하면 기본 정렬(오름차순)로 정렬된다.

-   기본 정렬

```
import java.util.Arrays;
import java.util.List;

public class IntermediateOperationExample {
    public static void main(String[] args) {
                // 이름을 모아둔 리스트 
        List<String> names = Arrays.asList("mason", "serena", "hana", "nana");

                // 인자값 없는 sort() 호출
        names.stream().sorted().forEach(System.out::println);
    }
}

// 출력값
hana
mason
nana
serena
```

-   역순으로 정렬

```
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

public class IntermediateOperationExample {
    public static void main(String[] args) {

        List<String> names = Arrays.asList("mason", "serena", "hana", nana");

                // 인자값에 Comparator 인터페이스에 규정된 메서드 사용
        names.stream().sorted(Comparator.reverseOrder()).forEach(System.out::println);

    }
}

// 출력값
serena
nana
mason
hana
```

#### 기타

-   skip()
    -   스트림의 일부 요소들을 건너뜀

```
import java.util.stream.IntStream;

public class IntermediateOperationExample {
    public static void main(String[] args) {

        // 1~10 범위의 정수로 구성된 스트림 생성
        IntStream intStream = IntStream.rangeClosed(1, 10);

        // 앞의 6개의 숫자를 건너뛰고 숫자 7부터 출력
        intStream.skip(6).forEach(System.out::println);
    }
}

// 출력값
7
8
9
10
```

-   limit()
    -   스트림의 일부를 자름

```
import java.util.stream.IntStream;

public class IntermediateOperationExample {
    public static void main(String[] args) {

        // 1~10 범위의 정수로 구성된 스트림 생성
        IntStream intStream = IntStream.rangeClosed(1, 10);

        // 앞에서부터 4개의 숫자만 출력
        intStream.limit(4).forEach(System.out::println);
    }
}

// 출력값
1
2
3
4
```

-   peek()
    -   forEach() 와 마찬가지로 요소들을 순회하며 특정 작업을 수행
    -   중간 연산자로 여러번 연결하여 사용 가능
    -   주로 코드의 에러를 찾기위한 디버깅 용도로 활용

```
import java.util.stream.IntStream;

public class IntermediateOperationExample {
    public static void main(String[] args) {

        // 요소들을 사용하여 IntStream 생성
        IntStream intStream = IntStream.of(1, 2, 2, 3, 3, 4, 5, 5, 7, 7, 7, 8);

        // 짝수만 필터링하여 합계 구하기
        int sum = intStream.filter(element -> element % 2 == 0)
                .peek(System.out::println)
                .sum();

        System.out.println("합계 = " + sum);
    }
}

// 출력값
2
2
4
8
합계 = 16
```

### 스트림의 최종 연산

> 스트림 파이프라인의 마지막 단계로 최종적으로 사용하고 나면, 해당 스트림은 닫히고 모든 연산이 종료된다.
> 
> 중간 연산은 최종 연산자가 수행될 떄야 스트림의 요소들이 중간 연산을 거쳐 가공된 후에 최종 연산에서 소모되는데 이를 "지연된 연산(lazy evaluation)"이라 부른다.

#### 기본 집계( sum(), count(), average(), max(), min() )

-   평균, 최대값, 최소값, 배열은 반환 타입이 Optional 객체로 일종의 래퍼 클래스
    -   null 값으로 인해서 NullPointerException 에러가 발생하는 현상을 객체 차원에서 효율적으로 방지하기 위한 목적으로 도입
    -   객체로 반환되는 값을 다시 기본형으로 변환하기 위해 사용되는 메서드가 추가된 것으로 스트림 파이프라인과 관계가 없음

```
import java.util.Arrays;

public class TerminalOperationExample {
    public static void main(String[] args) {
        // int형 배열 생성
        int[] intArray = {1,2,3,4,5};

        // 카운팅
        long count = Arrays.stream(intArray).count();
        System.out.println("intArr의 전체 요소 개수 " + count);

        // 합계
        long sum = Arrays.stream(intArray).sum();
        System.out.println("intArr의 전체 요소 합 " + sum);

        // 평균
        double average = Arrays.stream(intArray).average().getAsDouble();
        System.out.println("전체 요소의 평균값 " + average);

        // 최대값
        int max = Arrays.stream(intArray).max().getAsInt();
        System.out.println("최대값 " + max);

        // 최소값
        int min = Arrays.stream(intArray).min().getAsInt();
        System.out.println("최소값 " + min);

        // 배열의 첫 번째 요소 
        int first = Arrays.stream(intArray).findFirst().getAsInt();
        System.out.println("배열의 첫번째 요소 " + first);
    }
}

// 출력값
intArr의 전체 요소 개수 5
intArr의 전체 요소 합 15
전체 요소의 평균값 3.0
최대값 5
최소값 1
배열의 첫번째 요소 1
```

#### 매칭( allMatch(), anyMatch(), noneMatch() )

> 조건식 람다 Predicate 를 매개변수로 넘겨 스트림의 각 데이터 요소들이 특정한 조건을 충족하는 지 만족시키지 않는 지 검사하여, 그 결과를 boolean 값으로 반환한다.
> 
> match() 메서드는 크게 allMatch(), noneMatch(), noneMatch() 로 3가지 종류가 있다.
> 
> > -   allMatch() : 모든 요소들이 조건을 만족하는 지 여부를 판단
> > -   noneMatch() : 모든 요소들이 조건을 만족하지 않는 지 여부를 판단
> > -   anyMatch() : 하나라도 조건을 만족하는 요소가 있는 지 여부를 판단

```
import java.util.Arrays;

public class TerminalOperationExample {
    public static void main(String[] args) throws Exception {
        // int형 배열 생성
        int[] intArray = {3,6,9};

        // allMatch()
        boolean result = Arrays.stream(intArray).allMatch(element-> element % 3 == 0);
        System.out.println("요소 모두 3의 배수인가요? " + result);

        // anyMatch()
        result = Arrays.stream(intArray).anyMatch(element-> element % 2 == 0);
        System.out.println("요소 중 하나라도 2의 배수가 있나요? " + result);

        // noneMatch()
        result = Arrays.stream(intArray).noneMatch(element -> element % 4 == 0);
        System.out.println("요소 중 4의 배수가 하나도 없나요? " + result);
    }
}

// 출력값
요소 모두 3의 배수인가요? true
요소 중 하나라도 2의 배수가 있나요? true
요소 중 4의 배수가 하나도 없나요? false
```

#### 요소 소모( reduce() )

> 스트림의 요소를 줄여나가면서 연산을 수행하고 최종적인 결과를 반환한다.
> 
> 첫 번째와 두 번째 요소를 가지고 연산을 수행하고, 그 결과와 다음 세 번째 요소를 가지고 또다시 연산을 수행하는 식으로 연산이 끝날 때까지 반복한다.
> 
> 매개변수 타입은 BinaryOperator<T> 로 정의되어 있다.

```
import java.util.Arrays;

public class TerminalOperationExample {
    public static void main(String[] args) throws Exception {
        int[] intArray = {1,2,3,4,5};

        // sum()
        long sum = Arrays.stream(intArray).sum();
        System.out.println("intArray 전체 요소 합: " + sum);

        // 초기값이 없는 reduce()
        int sum1 = Arrays.stream(intArray)
                .map(element -> element * 2)
                    .reduce((a , b) -> a + b)
                .getAsInt();
        System.out.println("초기값이 없는 reduce(): " + sum1);

        // 초기값이 있는 reduce()
        int sum2= Arrays.stream(intArray)
                .map(element -> element * 2)
                .reduce(5, (a ,b) -> a + b);
        System.out.println("초기값이 있는 reduce(): " + sum2);
    }
}

// 출력값
intArray 전체 요소 합: 15
초기값이 없는 reduce(): 30
초기값이 있는 reduce(): 35
```

#### 요소 수집( collect() )

> 중간 연산을 통한 요소들의 데이터 가공 후 요소들을 수집하는 최종 처리 메서드이다.
> 
> 스트림의 요소들을 List, Set, Map 등 다른 타입의 결과로 수집하고 싶은 경우 사용할 수 있다.
> 
> collect() 메서드는 Collector 인터페이스 타입의 인자를 받아서 처리할 수 있는데 직접 구현하거나 미리 제공된 것들을 사용할 수 있고, 빈번하게 사용되는 기능들은 Collectors 클래스에서 제공하고 있다.
> 
> 요소를 수집하는 기능 이 외에도 요소 그룹핑 및 분할 등 다른 기능들을 제공한다.

```
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class TerminalOperationExample {

    public static void main(String[] args) {
        // Student 객체로 구성된 배열 리스트 생성 
        List<Student> totalList = Arrays.asList(
                new Student("serena", 100, Student.Gender.Female),
                new Student("hana", 80, Student.Gender.Female),
                new Student("mason", 90, Student.Gender.male),
                new Student("nana", 60, Student.Gender.Female)
        );

        // 스트림 연산 결과를 Map으로 반환
        Map<String, Integer> maleMap = totalList.stream()
                .filter(s -> s.getGender() == Student.Gender.Male)
                .collect(Collectors.toMap(
                        student -> student.getName(), // Key
                        student -> student.getScore() // Value
                ));

        // 출력
        System.out.println(maleMap);
    }
}

class Student {
    public enum Gender {Male, Female};
    private String name;
    private int score;
    private Gender gender;

    public Student(String name, int score, Gender gender) {
        this.name = name;
        this.score = score;
        this.gender = gender;
    }

    public String getName() {
        return name;
    }

    public int getScore() {
        return score;
    }

    public Gender getGender() {
        return gender;
    }
}

// 출력값
{mason=90}
```
