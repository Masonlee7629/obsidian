태그: #Java 
연결문서: [Java 기초 - 5](Java%20기초%20-%205.md)

## Java의 제어문

> Java의 조건문과 반복문을 통틀어 제어문이라고 한다

-   제어문
    -   조건문 : if문, switch문
    -   반복문 : for문, while문, do-while문

### Java의 조건문

> 특정 조건에 부합하는 경우, 특정 코드를 실행 또는 실행시키지 않을 수 있다

-   if문
    -   소괄호 안에 boolean값으로 평가될 수 있는 조건식을 넣고 실행 블록인 중괄호에는 조건식이 참일 때 실행할 코드를 입력

```
if(조건식) {
    // 조건식이 참일 떄 실행될 블록
}
```

-   if ~ else문
    -   if문의 조건식이 true일 경우 해당 블록이 실행되고 조건식이 false라면 다음 조건문인 else if문의 조건식을 확인하고 else if문의 모든 조건식이 false일 때 else블록을 실행

```
if(조건식1) {
  // 조건식1이 참일 때 실행되는 블록
}
else if(조건식2) {
  // 조건식1이 거짓이고 조건식 2가 참일 때 실행되는 블록
}
else {
  // 조건식1, 2 모두 거짓일 때 실행되는 블록으로 생략 가능
}
```

-   switch문
    -   변수가 값에 따라 실행문이 선택되므로 처리할 경우의 수가 많은 경우 if문보다 효율적
    -   조건식의 결과는 정수 또는 문자열이어야 하며 case문의 값을 정수, 상수만 가능하고 중복 불가
    -   break문은 각 case문의 영역을 구분하는 역할을 하며 경우에 따라 고의적으로 생략

```
switch(조건식) {
  case value1 :
    // 조건식의 결과가 value1과 같을 때 실행되는 블록
    Syetem.out.println("Hello");
    break;
  case value2 :
    // 조건식의 결과가 value2와 같을 때 실행되는 블록
    Syetem.out.println("Java");
    break;
  default :
    // 조건식의 결과와 일치하는 case문이 없을 때 실행되는 블록
    Syetem.out.println("Hello Java");
}
```

### Java의 반복문

> 특정 코드를 반복적으로 실행시킬 수 있다

-   for문
    -   조건식이 참인 동안 주어진 횟수만큼 실행문을 반복적으로 실행

```
public class Main {
  public static void main(String[] args) {    
    int tmp = 0;
    for(int i = 0; i < 10; i++) {  // 초기화; 조건식; 증감식
      tmp += i;
    }
    System.out.println(tmp);      // 1~9의 합인 45를 출력한다.
  }
}
```

-   while문
    -   조건식이 true일 경우 계속해서 반복
    -   조건식에 true를 사용하면 무한 루프를 돌기 때문에 while문에 탈출 코드가 필요함

```
int num = 0, sum = 0; // 초기화
while(num <= 11){     // 조건식
  sum += num;         // 조건식이 참인 경우 계속 실행되는 블록
  num ++;             // 증감식
}
System.out.println(sum);
```

-   do-while문
    -   실행문을 처음 한 번은 무조건 실행한 후, 조건식에 따라 while문을 실행

```
do {
    // 처음 한 번은 무조건 실행되는 블록
} while(조건식);  // 끝에 ';'를 잊지 않도록 주의
```

-   break문
    -   자신이 포함된 가장 가까운 반복문을 벗어남

```
public class Main{
  public static void main(String[] args){
    int sum = 0, i = 0;

    while(true) {
      if(sum > 100)
        break;  // break문이 실행되면 break문 아래 코드를 실행하지 않고 while문을 탈출
      ++i;
      sum += i;
    }
    System.out.println("sum = " + sum);
  }
}
```

-   continue문
    -   반복문 내에서만 사용되며 반복문 실행 중 continue문을 만나면 다음 차례의 반복을 수행
    -   반복 중 특정 조건을 만족하는 경우를 제외할 때 유용함

```
public class Main{
  public static void main(String[] args) {
    for(int i = 0; i <= 10; i++) {
      if(i % 2 == 0)
        continue; // 조건식이 참일 때 실행되며 블록 끝으로 이동하여 다음 반복을 실행
      Syetem.out.println(i);
    }
  }
}
```