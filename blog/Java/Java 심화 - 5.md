태그: #Java 
연결문서: [Java 심화 - 6](Java%20심화%20-%206.md)

# Java 심화

## 스레드(Thread)

> 애플리케이션이 실행되면 운영체제가 해당 애플리케이션에게 메모리를 할당해주며 애플리케이션이 실행되는데, 이처럼 실행 중인 애플리케이션을 프로세스라고 하며, 프로세스 내에서 실행되는 소스 코드의 실행 흐름을 스레드라고 한다.
> 
> 단 하나의 스레드를 가지는 프로세스를 싱글 스레드 프로세스, 여러 개의 스레드를 가지는 프로세스를 멀티 스레드 프로세스라고 한다.
> 
> 프로세스는 데이터, 컴퓨터 자원, 스레드로 구성되며, 스레드는 데이터와 애플리케이션이 확보한 자원을 활용하여 소스 코드를 실행한다.

#### 메인 스레드(Main thread)

> 자바 애플리케이션을 실행하면 가장 먼저 실행되는 메서드는 main 메서드이며, 메인 스레드가 main 메서드를 실행시켜준다. 메인 스레드는 main 메서드의 코드를 처음부터 끝까지 순차적으로 실행시키며, 코드의 끝을 만나거나 return문을 만나면 실행을 종료한다.

-   싱글 스레드 프로세스
    -   애플리케이션이 실행되어 프로세스가 될 때 오로지 메인 스레드만 가진 것  
        [##_Image|kage@cmeLn9/btsaqP8ZkwU/D2EK5udRX7H5WOZUSgjfX1/img.png|CDM|1.3|{"originWidth":0,"originHeight":0,"style":"alignCenter","width":null}_##]

-   멀티 스레스 프로세스
    -   하나의 프로세스가 여러 개의 스레드를 가진 것
    -   여러 개의 스레드를 가진다는 것은 여러 스레드가 동시에 작업을 수행할 수 있음을 의미하여 **멀티 스레딩**이라고 함
    -   멀티 스레딩은 하나의 애플리케이션 내에서 여러 작업을 동시에 수행하는 멀티 테스킹을 구현하는 데에 핵심적인 역할을 수행  
        [##_Image|kage@bQkURU/btsak2A7C5J/bIAOrvZbf2sK5rIOwppUMk/img.png|CDM|1.3|{"originWidth":0,"originHeight":0,"style":"alignCenter","width":null}_##]

### 스레드의 생성과 실행

#### 작업 스레드 생성과 실행

> 메인 스레드 외에 별도의 작업 스레드를 활용한다는 것은 작업 스레드가 수행할 코드를 작성하고, 작업 스레드를 생성하여 실행시키는 것을 의미한다.
> 
> 자바는 객체지향 언어로 모든 자바 코드는 클래스 안에 작성되기 때문에 스레드가 수행할 코드도 클래스 내부에 작성해주어야 하며, run() 이라는 메서드 내에 스레드가 처리할 작업을 작성하도록 규정되어 있다.
> 
> run() 메서드는 Runnable 인터페이스와 Thread 클래스에 정의되어 있고, 작업 스레드를 생성하고 실행하는 방법은 두 가지가 있다.
> 
> > -   첫 번째 방법
> >     -   Runnable 인터페이스를 구현한 객체에서 run() 을 구현하여 스레드를 생성하고 실행하는 방법
> > -   두 번째 방법
> >     -   Thread 클래스를 상속 받은 하위 클래스에서 run() 을 구현하여 스레드를 생성하고 실행시키는 방법

#### Runnable 인터페이스를 구현한 객체에서 run() 을 구현하여 스레드를 생성하고 실행하는 방법

-   Runnable 인터페이스를 구현한 객체를 생성
    -   임의의 클래스를 만들고, Runnable을 구현하며, Runnable에는 run() 이 정의되어 있기 때문에 반드시 run() 을 구현해야 함

```
public class ThreadExample1 {
    public static void main(String[] args) {

    }
}

// Runnable 인터페이스를 구현하는 클래스
class ThreadTask1 implements Runnable {
    public void run() {

    }
}
```

-   run() 의 메서드 바디에 새롭게 생성된 작업 스레드가 수행할 코드 작성

```
public class ThreadExample1 {
    public static void main(String[] args) {

    }
}

class ThreadTask1 implements Runnable {

    // run() 메서드 바디에 스레드가 수행할 작업 내용 작성
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

-   스레드 생성
    -   Runnable 인터페이스를 구현한 객체를 활용하여 스레드를 생성할 때에는 Runnable 구현 객체를 인자로 전달하면서 Thread 클래스를 인스턴스화

```
public class ThreadExample1 {
    public static void main(String[] args) {

        // Runnable 인터페이스를 구현한 객체 생성
        Runnable task1 = new ThreadTask1();

        // Runnable 구현 객체를 인자로 전달하면서 Thread 클래스를 인스턴스화하여 스레드를 생성
        Thread thread1 = new Thread(task1);

        // 위의 두 줄을 아래와 같이 한 줄로 축약할 수도 있습니다. 
        // Thread thread1 = new Thread(new ThreadTask1());

    }
}

class ThreadTask1 implements Runnable {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

-   run() 메서드 내부의 코드를 실행시키기 위해 start() 메서드를 호출

```
public class ThreadExample1 {
    public static void main(String[] args) {
        Runnable task1 = new ThreadTask1();
        Thread thread1 = new Thread(task1);

        // 작업 스레드를 실행시켜, run() 내부의 코드를 처리하도록 합니다. 
        thread1.start();
    }
}

class ThreadTask1 implements Runnable {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

-   main 메서드에 반복문 추가 후, 코드 실행

```
public class ThreadExample1 {
    public static void main(String[] args) {
        Runnable task1 = new ThreadTask1();
        Thread thread1 = new Thread(task1);

        thread1.start();

        // 반복문 추가
        for (int i = 0; i < 100; i++) {
            System.out.print("@");
        }
    }
}

class ThreadTask1 implements Runnable {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

-   출력 결과
    -   @는 main 메서드의 반복문에서 출력한 문자로 메인 스레드의 반복문 코드 실행에 의해 출력
    -   \#는 run() 메서드의 반복문에서 출력한 문자로 작업 스레드의 반복문 코드 실행에 의해 출력
    -   메인 스레드와 작업 스레드가 동시에 병렬로 실행되면서 각각 main 메서드와 run() 메서드의 코드를 실행시켰기 때문에 두 가지 문자가 섞여서 출력됨

```
@@@@@@@@@@@######@@@@@############################
@#########@@@@@@@@@@@@@@@@############@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@##@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@###########################################
Process finished with exit code 0
```

#### Thread 클래스를 상속 받은 하위 클래스에서 run()을 구현하여 스레드를 생성하고 실행하는 방법

-   Thread 클래스를 상속 받는 하위 클래스 생성
    -   Thread 클래스에는 run() 메서드가 정의되어 있어며, run() 메서드를 오버라이딩해야 함

```
public class ThreadExample2 {
    public static void main(String[] args) {

    }
}

// Thread 클래스를 상속받는 클래스 작성
class ThreadTask2 extends Thread {
    public void run() {

    }
}
```

-   run() 메서드 바디에 새롭게 생성될 스레드가 수행할 적업을 작성

```
public class ThreadExample2 {
    public static void main(String[] args) {

    }
}

class ThreadTask2 extends Thread {

    // run() 메서드 바디에 스레드가 수행할 작업 내용 작성
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

-   run() 내의 코드를 실행할 스레드를 생성
    -   Thread 클래스를 직접 인스턴스화하지 않음

```
public class ThreadExample2 {
    public static void main(String[]args) {

        // Thread 클래스를 상속받은 클래스를 인스턴스화하여 스레드를 생성
        ThreadTask2 thread2 = new ThreadTask2();
    }
}

class ThreadTask2 extends Thread {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

-   start() 메서드를 실행하고, main 메서드에 반복문 추가 후, 코드 실행

```
public class ThreadExample2 {
    public static void main(String[] args) {

        ThreadTask2 thread2 = new ThreadTask2();

        // 작업 스레드를 실행시켜, run() 내부의 코드를 처리하도록 합니다. 
        thread2.start();

        // 반복문 추가
        for (int i = 0; i < 100; i++) {
            System.out.print("@");
        }
    }
}

class ThreadTask2 extends Thread {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.print("#");
        }
    }
}
```

#### 익명 객체를 사용하여 스레드 생성하고 실행하기

-   Runnable 익명 구현 객체를 활용한 스레드 생성 및 실행

```
public class ThreadExample1 {
    public static void main(String[] args) {

        // 익명 Runnable 구현 객체를 활용하여 스레드 생성
        Thread thread1 = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 100; i++) {
                    System.out.print("#");
                }
            }
        });

        thread1.start();

        for (int i = 0; i < 100; i++) {
            System.out.print("@");
        }
    }
}
```

-   Thread 익명 하위 객체를 활용한 스레드 생성 및 실행

```
public class ThreadExample2 {
    public static void main(String[] args) {

        // 익명 Thread 하위 객체를 활용한 스레드 생성
        Thread thread2 = new Thread() {
            public void run() {
                for (int i = 0; i < 100; i++) {
                    System.out.print("#");
                }
            }
        };

        thread2.start();

        for (int i = 0; i < 100; i++) {
            System.out.print("@");
        }
    }
}
```

### 스레드의 이름

> 메인 스레드는 "main"이라는 이름을 가지며, 그 외에 추가적으로 생성한 스레드는 기본적으로 "Thread-n"이라는 이름을 가진다.

#### 스레드의 이름 조회하기

-   스레드의 이름은 스레드의\_참조값.getName() 으로 조회

```
public class ThreadExample3 {
    public static void main(String[] args) {

        Thread thread3 = new Thread(new Runnable() {
            public void run() {
                System.out.println("Get Thread Name");
            }
        });

        thread3.start();

        System.out.println("thread3.getName() = " + thread3.getName());
    }
}

// 출력 결과
Get Thread Name
thread3.getName() = Thread-0
```

#### 스레드의 이름 설정하기

-   스레드의 이름은 스레드의\_참조값.setName() 으로 설정

```
public class ThreadExample4 {
    public static void main(String[] args) {

        Thread thread4 = new Thread(new Runnable() {
            public void run() {
                System.out.println("Set And Get Thread Name");
            }
        });

        thread4.start();

        System.out.println("thread4.getName() = " + thread4.getName());

        thread4.setName("Code States");

        System.out.println("thread4.getName() = " + thread4.getName());
    }
}

// 출력 결과
Set And Get Thread Name
thread4.getName() = Thread-0
thread4.getName() = Code States
```

#### 스레드 인스턴스의 주소값 얻기

-   Thread 클래스의 정적 메서드인 currentThread() 를 사용
    -   스레드의 이름을 조회하고 설정하는 두 메서드는 Thread 클래스로부터 인스턴스화된 인스턴스의 메서드이므로 호출할 떄에 스레드 객체의 참조가 필요

```
public class ThreadExample1 {
    public static void main(String[] args) {

        Thread thread1 = new Thread(new Runnable() {
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });

        thread1.start();
        System.out.println(Thread.currentThread().getName());
    }
}

// 출력 결과
main
Thread-0
```

### 스레드의 동기화

> 멀티 스레드 프로세스에서 여러 스레드가 동일한 데이터를 공유하게 되어 발생하는 문제를 방지할 수 있다.
> 
> 스레드의 동기화를 적용하려면 임계 영역과 락(Lock)에 대한 이해가 필요하다.

#### 두 스레드가 동일한 데이터를 공유하게 되어 발생할 수 있는 문제

-   두 스레드 간에 객체가 공유되기 때문에 발생하는 오류
    -   오류 1. 인출금과 잔액이 제대로 출력되지 못함
    -   오류 2. 음수의 잔액이 발생
    -   오류 3. 인출에 실해한 경우 -> DENIED 가 제대로 출력되지 않음

```
public class ThreadExample3 {
    public static void main(String[] args) {

        Runnable threadTask3 = new ThreadTask3();
        Thread thread3_1 = new Thread(threadTask3);
        Thread thread3_2 = new Thread(threadTask3);

        thread3_1.setName("김코딩");
        thread3_2.setName("박자바");

        thread3_1.start();
        thread3_2.start();
    }
}

class Account {

    // 잔액을 나타내는 변수
    private int balance = 1000;

    public int getBalance() {
        return balance;
    }

    // 인출 성공 시 true, 실패 시 false 반환
    public boolean withdraw(int money) {

        // 인출 가능 여부 판단 : 잔액이 인출하고자 하는 금액보다 같거나 많아야 합니다.
        if (balance >= money) {

            // if문의 실행부에 진입하자마자 해당 스레드를 일시 정지 시키고, 
            // 다른 스레드에게 제어권을 강제로 넘깁니다.
            // 일부러 문제 상황을 발생시키기 위해 추가한 코드입니다.
            // 어떤 스레드가 일시 정지되면, 대기열에서 기다리고 있던 다른 스레드가 실행됩니다.
            try { Thread.sleep(1000); } catch (Exception error) {}

            // 잔액에서 인출금을 깎아 새로운 잔액을 기록합니다.
            balance -= money;

            return true;
        }
        return false;
    }
}

class ThreadTask3 implements Runnable {
    Account account = new Account();

    public void run() {
        while (account.getBalance() > 0) {

            // 100 ~ 300원의 인출금을 랜덤으로 정합니다. 
            int money = (int)(Math.random() * 3 + 1) * 100;

            // withdraw를 실행시키는 동시에 인출 성공 여부를 변수에 할당합니다. 
            boolean denied = !account.withdraw(money);

            // 인출 결과 확인
            // 만약, withraw가 false를 리턴하였다면, 즉 인출에 실패했다면,
            // 해당 내역에 -> DENIED를 출력합니다. 
            System.out.println(String.format("Withdraw %d₩ By %s. Balance : %d %s",
                    money, Thread.currentThread().getName(), account.getBalance(), denied ? "-> DENIED" : "")
            );
        }
    }
}

// 출력 결과(매 실행 시마다 다를 수 있음)
Withdraw 100₩ By 김코딩. Balance : 600 
Withdraw 300₩ By 박자바. Balance : 600 
Withdraw 200₩ By 김코딩. Balance : 400 
Withdraw 200₩ By 박자바. Balance : 200 
Withdraw 200₩ By 김코딩. Balance : -100 
Withdraw 100₩ By 박자바. Balance : -100 
```

#### 임계 영역(Critical section)과 락(Lock)

> **임계 영역은 오로지 하나의 스레드만 코드를 실행할 수 있는 코드 영역을 의미하며, 락은 임계 영역을 포함하고 있는 객체에 접근할 수 있는 권한을 의미**한다.
> 
> 즉, 임계 영역으로 설정된 객체가 다른 스레드에 의해 작업이 이루어지고 있지 않을 때, 임의의 스레드 A는 해당 객체에 대한 락을 획득하여 임계 영역 내의 코드를 실행할 수 있다.
> 
> 이 때, 스레드 A가 임계 영역 내의 코드를 실행 중일 때는 다른 스레드들은 락이 없으므로 이 객체의 임계 영역 내의 코드를 실행할 수 없다.
> 
> 스레드 A가 임계 영역 내의 코드를 모두 실행하면 락을 반납하고, 이 때부터 다른 스레드들 중 하나가 락을 획득하여 임계 영역 내의 코드를 실행할 수 있다.
> 
> 두 스레드가 동일한 데이터를 공유하게 되어 발생할 수 있는 문제에서 필요했던 것은 두 스레드가 동시에 실행하면 안되는 영역을 설정하는 것이다.
> 
> 특정 코드 구간을 임계 영역으로 설정할 때에는 synchronized라는 키워드를 사용하며, synchronized 키워드는 **메서드 전체를 임계 영역으로 지정하는 것**과 **특정한 영역을 임계 영역으로 지정하는 것**으로 두 가지 방법을 사용할 수 있다.

-   메서드 전체를 임계 영역으로 지정하기
    -   메서드의 반환 타입 좌측에 synchronized 키워드를 작성
    -   메서드 전체를 임계 영역으로 지정하면 메서드가 호출되었을 때, 메서드를 실행할 스레드는 메서드가 포함된 객체의 락을 얻음

```
class Account {
    ...
    public synchronized boolean withdraw(int money) {
        if (balance >= money) {
            try { Thread.sleep(1000); } catch (Exception error) {}
            balance -= money;
            return true;
        }
        return false;
    }
}
```

-   특정한 영역을 임계 영역으로 지정하기
    -   synchronized 키워드와 함께 소괄호 안에 해당 영역이 포함된 객체의 참조를 넣고, 중괄호 블럭을 열어, 블럭 내에 코드를 작성
    -   코드를 실행하고 있는 스레드가 소괄호 안에 해당하는 객체의 락을 얻고, 배타적으로 임계 영역 내의 코드를 실행

```
class Account {
    ...
    public boolean withdraw(int money) {
            synchronized (this) {
                if (balance >= money) {
                    try { Thread.sleep(1000); } catch (Exception error) {}
                    balance -= money;
                    return true;
                }
                return false;
            }
    }
}
```

### 스레드의 상태와 실행 제어

> 스레드를 생성한 후에 실행시키기 위해서는 start() 메서드를 호출해야한다고 했지만, 엄밀히 말하면 start() 는 스레드를 실행시키는 메서드가 아니다.
> 
> start() 는 스레드의 상태를 **실행 대기 상태**로 만들어주는 메서드이며, 어떤 스레드가 start() 에 의해 실행 대기 상태가 되면 운영체제가 적절할 때에 스레드를 실행시켜준다.

#### 스레드의 상태와 실행 제어 메서드 요약

[##_Image|kage@pFZqQ/btsaip4RTxu/mBxdZQfbwA9I3IIhA4p6EK/img.png|CDM|1.3|{"originWidth":0,"originHeight":0,"style":"alignCenter","width":null}_##]

#### 스레드 실행 제어 메서드

-   sleep(long milliSecond) : milliSecond 동안 스레드를 잠시 멈춤
    -   sleep() 은 Thread의 클래스 메서드로 호출할 때에는 Thread.sleep(1000); 과 같이 클래스를 통해서 호출하는 것을 권장
    -   sleep() 을 호출하면 sleep() 을 호출하는 코드를 실행한 스레드의 상태가 **실행 상태에서 일시정지 상태로 전환**
    -   인자로 전달된 시간 만큼의 시간이 경과하거나 interrupt() 를 호출한 경우 **실행 대기 상태로 복귀**

```
try { Thread.sleep(1000); } catch (Exception error) {}
```

-   interrupt() : 일시 중지 상태인 스레드를 실행 대기 상태로 복귀
    -   기본적으로 예외가 발생하기 때문에 try...catch 문을 사용해서 예외 처리 필요
    -   멈춰 있는 스레드가 아닌 다른 스레드에서 멈춰\_있는\_스레드.interrupt() 를 호출하면 기존에 호출되어 스레드를 멈추게 했던 sleep(), wait(), join() 메서드에서 예외가 발생되며, 그에 따라 일시 정지가 풀림

```
public class ThreadExample5 {
    public static void main(String[] args) {
        Thread thread1 = new Thread() {
            public void run() {
                try {
                    while (true) Thread.sleep(1000);
                }
                catch (Exception e) {}
                System.out.println("Woke Up!!!");
            }
        };

        System.out.println("thread1.getState() = " + thread1.getState());

        thread1.start();

        System.out.println("thread1.getState() = " + thread1.getState());

        while (true) {
            if (thread1.getState() == Thread.State.TIMED_WAITING) {
                System.out.println("thread1.getState() = " + thread1.getState());
                break;
            }
        }

        thread1.interrupt();

        while (true) {
            if (thread1.getState() == Thread.State.RUNNABLE) {
                System.out.println("thread1.getState() = " + thread1.getState());
                break;
            }
        }

        while (true) {
            if (thread1.getState() == Thread.State.TERMINATED) {
                System.out.println("thread1.getState() = " + thread1.getState());
                break;
            }
        }
    }
}

// 출력 결과
thread1.getState() = NEW
thread1.getState() = RUNNABLE
thread1.getState() = TIMED_WAITING
Woke Up!!!
thread1.getState() = RUNNABLE
thread1.getState() = TERMINATED
```

-   yield() : 다른 스레드에게 실행을 양보
    -   운영 체제의 스케줄러에 의해 할당 받은 실행 시간을 다른 스레드에게 양보
    -   아래 예제와 같이 example의 값이 false라면 스레드는 while문의 반복이 불필요함에도 계속해서 반복하는데, yield() 를 호출하면 example의 값이 false일 때에 while문의 반복을 멈추고 실행 대기 상태로 바뀌며, 자신에게 남은 실행 시간을 실행 대기열 상 우선순위가 높은 다른 스레드에게 양보

```
public void run() {
        while (true) {
                if (example) {
                        ...
                }
                else Thread.yield();
        }
}
```

-   join() : 다른 스레드의 작업이 끝날 때까지 대기
    -   특정 스레드가 작업하는 동안에 자신을 일시 중지 상태로 만드는 메서드
    -   인자로 시간을 밀리초 단위로 전달할 수 있으며, 전달한 인자만큼의 시간이 경과하거나, interrupt() 가 호출되거나, join() 호출 시 지정했던 다른 스레드가 모든 작업을 마치면 다시 실행 대기 상태로 복귀
    -   join()과 sleep() 의 유사점
        -   일시 중지 상태가 됨
            -   try...catch문으로 감싸서 사용
            -   interrupt() 에 의해 실행 대기 상태로 복귀 가능
    -   join()과 sleep() 의 차이점
        
        ```
         - sleep() 은 Thread 클래스의 static 메서드이고, join() 은 특정 스레드에 대해 동작하는 인스턴스 메서드임
        ```
        
        -   Ex) Thread.sleep(1000);
        -   Ex) Thread.sleep(1000);

```
public class ThreadExample {
    public static void main(String[] args) {
        SumThread sumThread = new SumThread();

        sumThread.setTo(10);

        sumThread.start();

        // 메인 스레드가 sumThread의 작업이 끝날 때까지 기다립니다. 
        try { sumThread.join(); } catch (Exception e) {}

        System.out.println(String.format("1부터 %d까지의 합 : %d", sumThread.getTo(), sumThread.getSum()));
    }
}

class SumThread extends Thread {
    private long sum;
    private int to;

    public long getSum() {
        return sum;
    }

    public int getTo() {
        return to;
    }

    public void setTo(int to) {
        this.to = to;
    }

    public void run() {
        for (int i = 1; i <= to; i++) {
            sum += i;
        }
    }
}
```

-   wait(), notify() : 스레드 간 협업에 사용
    -   두 스레드가 교대로 작업을 처리해야할 때 사용할 수 있는 메서드
    -   스레드A와 스레드B의 협업 플로우
        -   스레드 A가 공유 객체에 자신의 작업을 완료하고 스레드 B와 교대하기 위해 notify() 를 호출
            -   notify() 가 호출되면 스레드 B가 실행 대기 상태가 되며, 곧 실행되고 이어서 스레드 A는 wait() 를 호출하며 자기 자신을 일시 정지 상태로 만듦
            -   이후 스레드 B가 작업을 완료하면 notify() 를 호출하여 작업을 중단하고 있던 스레드 A를 다시 실행 대기 상태로 복귀시킨 후, wait() 를 호출하여 자기 자신의 상태를 일시 정지 상태로 전환
            -   위 과정을 반복하며 두 스레드는 공유 객체에 대해 서로 대타적으로 접근하며 효과적으로 협업 가능

```
public class ThreadExample5 {
    public static void main(String[] args) {
        WorkObject sharedObject = new WorkObject();

        ThreadA threadA = new ThreadA(sharedObject);
        ThreadB threadB = new ThreadB(sharedObject);

        threadA.start();
        threadB.start();
    }
}

class WorkObject {
    public synchronized void methodA() {
        System.out.println("ThreadA의 methodA Working");
        notify();
        try { wait(); } catch(Exception e) {}
    }

    public synchronized void methodB() {
        System.out.println("ThreadB의 methodB Working");
        notify();
        try { wait(); } catch(Exception e) {}
    }
}

class ThreadA extends Thread {
    private WorkObject workObject;

    public ThreadA(WorkObject workObject) {
        this.workObject = workObject;
    }

    public void run() {
        for(int i = 0; i < 10; i++) {
            workObject.methodA();
        }
    }
}

class ThreadB extends Thread {
    private WorkObject workObject;

    public ThreadB(WorkObject workObject) {
        this.workObject = workObject;
    }

    public void run() {
        for(int i = 0; i < 10; i++) {
            workObject.methodB();
        }
    }
}
```
