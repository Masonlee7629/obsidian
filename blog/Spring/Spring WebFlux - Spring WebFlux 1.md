태그: #Spring 
연결문서: [Spring WebFlux - Spring WebFlux 2](Spring%20WebFlux%20-%20Spring%20WebFlux%202.md)

# Spring WebFlux

## Spring WebFlux란?
- Spring 5 버전에 새롭게 추가된 기술 스택은 바로 리액티브(Reactive) 스택이며, 리액티브(Reactive) 스택이 Spring 5에 추가되면서 항상 언급되는 기술이 바로 WebFlux
- WebFlux라는 용어는 Reactor의 타입인 Flux가 Web에서 사용된다고 할 수 있으며, 조금 더 넓게 생각해 보면 WebFlux는 리액티브 한 웹 애플리케이션을 구현하기 위한 기술 자체를 상징하고 있다고 할 수 있음
- Spring WebFlux는 Spring 5부터 지원하는 리액티브 웹 애플리케이션을 위한 웹 프레임워크
    - Spring MVC 프레임워크를 사용해서 웹 애플리케이션을 구현할 수 있듯이 Spring WebFlux 프레임워크를 사용해서 리액티브 한 웹 애플리케이션을 구현할 수 있음
    - Spring WebFlux에서 꼭 Reactor만 사용할 수 있는 것은 아
    - WebFlux가 리액티브 스트림즈(Reactive Streams)의 사양인 인터페이스를 기반으로 동작하기 때문에 리액티브 스트림즈를 구현한 구현체라면 대부분 Reactor 대신 사용할 수 있음
    - Reactor 이외의 리액티브 라이브러리는 ReactiveAdapter와 ReactiveAdapterRegistry를 통해 사용 가능함

<br>

### Spring WebFlux 애플리케이션 vs Spring MVC 애플리케이션

#### 기술 스택 비교
- Spring WebFlux의 기술적인 특징을 이해하기 위해서는 Spring MVC의 기술 스택과 비교해 보는 것이 가장 좋음
<br>

![](image_42.png)

<br>

- (1) Spring WebFlux의 경우 Non-Blocking 통신을 지원하지만 Spring MVC의 경우 Non-Blocking이 아닌 Blocking 통신 방식을 사용함
- (2) Spring WebFlux의 경우 Reactive Adapter를 사용해서 Reactor 뿐만 아니라 RxJava 등의 다른 리액티브 라이브러리를 사용할 수 있는 유연함을 제공하는 반면, Spring MVC의 경우 Servlet API의 스펙에 의존적
- (3) Spring WebFlux와 Spring MVC 모두 보안을 적용하기 위해서 Spring Security를 사용하는데, Spring WebFlux의 경우 서블릿 필터 방식이 아닌 WebFilter를 사용해 리액티브 특성에 맞게 인증과 권한 등의 보안을 적용함
- (4) Reactive Stack의 경우, 웹 계층(프레젠테이션 계층, API 계층)에서는 Spring WebFlux를 사용하며 Servlet Stack의 경우, Spring MVC를 사용함
- (5) Spring WebFlux의 경우 완전한 Non-Blocking 통신을 위해 리액티브 스택을 데이터 액세스 계층까지 확장하며, R2DBC(Reactive Relation Database Connectivity)는 관계형 데이터베이스에 Non-Blocking 통신을 적용하기 위한 표준 사양(Specification)이며, MySQL, Oracle 등의 데이터베이스 벤더에서는 R2DBC 사양에 맞는 드라이버를 구현해서 공급함

<br>

#### 벤 다이어그램을 통한 비교
- Spring MVC와 Spring WebFlux는 전혀 다른 기술임에도 불구하고, 두 기술을 사용하는 입장에서는 최소한의 변경 포인트만 유지하면서 일관된 접근 방식으로 두 기술을 사용할 수 있도록 구성됨
<br>

![](image_43.png)

<br>

- @Controleller(또는 @RestController) 같은 애너테이션을 Spring WebFlux에서 사용할 수 있으며, RestTemplate을 대체할 수 있는 WebClient를 Spring MVC와 Spring WebFlux에서 모두 사용할 수 있음
- Tomcat, Jetty 등의 서블릿 컨테이너는 서블릿 방식과 리액티브 방식의 통신을 모두 지원하고 있음

<br>

#### 샘플 코드로 보는 Spring MVC와 Spring WebFlux

![](image_44.png)

<br>

- Spring MVC 샘플 애플리케이션과 Spring WebFlux 샘플 애플리케이션 모두 같은 구조로 구성됨
- 애플리케이션이 두 개이므로 애플리케이션 구현 코드 역시 IntelliJ IDE 상에서 각각의 프로젝트에서 구현됨

<br>

##### Spring MVC의 Blocking 처리 방식

```java
// Spring MVC 기반 메인 애플리케이션의 Controller 샘플 코드
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.time.LocalDateTime;

@Slf4j
@RestController
@RequestMapping("/coffees")
public class SpringMvcMainCoffeeController {
    private final RestTemplate restTemplate;

    String uri = "http://localhost:7070/coffees/1";

    public SpringMvcMainCoffeeController(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    @GetMapping("/{coffee-id}")
    public ResponseEntity getCoffee(@PathVariable("coffee-id") long coffeeId) {
        log.info("# call Spring MVC Main Controller: {}", LocalDateTime.now());
        
        // (1)  
        ResponseEntity<CoffeeResponseDto> response = restTemplate.getForEntity(uri, CoffeeResponseDto.class);
        return ResponseEntity.ok(response.getBody());
    }
}
```

- (1)에 알 수 있다시피 클라이언트의 요청을 메인 애플리케이션에서 직접 처리하는 것이 아니라 Spring의 Rest Client인 RestTemplate을 이용해서 외부에 있는 다른 애플리케이션에게 한번 더 요청을 전송함
- 애플리케이션을 실제로 실행시키면 메인 애플리케이션은 8080 포트에서 실행되며, 외부 애플리케이션은 7070 포트에서 실행됨
- 클라이언트 쪽에서 getCoffee() 핸들러 메서드로 요청을 전송하면 "# call Spring MVC Main Controller: {요청 수신 시간}"와 같이 로그가 출력됨

<br>

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/coffees")
public class SpringMvcOutboundCoffeeController {
    @GetMapping("/{coffee-id}")
    public ResponseEntity getCoffee(@PathVariable("coffee-id") long coffeeId) throws InterruptedException {
        CoffeeResponseDto responseDto = new CoffeeResponseDto(coffeeId, "카페라떼", "CafeLattee", 4000);

        // (1)
        Thread.sleep(5000);
        return ResponseEntity.ok(responseDto);
    }
}
```

- 메인 애플리케이션 Controller(SpringMvcMainCoffeeController)의 getCoffee() 핸들러 메서드에서 RestTemplate으로 전송한 요청을 전달받는 외부 애플리케이션의 Controller(SpringMvcOutboundCoffeeController) 코드
- Spring MVC 애플리케이션의 요청 처리 방식을 확인하는 것이 주목적이기 때문에 별도의 데이터베이스 연동 없이 단순히 Stub 데이터(responseDto)를 응답으로 넘겨주고 있음
- 외부 애플리케이션의 요청 처리 시간이 오래 걸리는 것을 시뮬레이션하기 위해서 (1)과 같이 요청 처리 스레드에 지연 시간을 주었음
    - Thread.sleep(5000)을 설정하면 하나의 요청이 들어오면 5초간 스레드의 동작이 멈춤

<br>

```java
import com.CoffeeResponseDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestTemplate;
import java.time.LocalTime;

@Slf4j
@SpringBootApplication
public class SpringMvcMainSampleApplication {
    private final RestTemplate restTemplate;

	public SpringMvcMainSampleApplication(RestTemplateBuilder restTemplateBuilder) {
		this.restTemplate = restTemplateBuilder.build();
	}
	public static void main(String[] args) {
		SpringApplication.run(SpringMvcMainSampleApplication.class, args);
	}

	@Bean
	public CommandLineRunner run() {
		return (String... args) -> {
			log.info("# 요청 시작 시간: {}", LocalTime.now());

                        // (1)
			for (int i = 1; i <= 5; i++) {
				CoffeeResponseDto response = this.getCoffee();
				log.info("{}: coffee name: {}", LocalTime.now(), response.getKorName());
			}
		};
	}

	private CoffeeResponseDto getCoffee() {
		String uri = "http://localhost:8080/coffees/1";
		ResponseEntity<CoffeeResponseDto> response = restTemplate.getForEntity(uri, CoffeeResponseDto.class);

		return response.getBody();
	}
}

// 실행 결과
INFO 17484 --- [           main] c.c.SpringMvcMainSampleApplication       : # 요청 시작 시간: xx:xx:xx.xxxxxxxxx
INFO 17484 --- [nio-8080-exec-1] c.c.c.SpringMvcMainCoffeeController      : # call Spring MVC Main Controller: xxxx-xx-xxT14:xx:xx.xxxxxxxxx  // (1-1) 첫 번째 요청
INFO 17484 --- [           main] c.c.SpringMvcMainSampleApplication       : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼 // (1-2) 첫 번째 요청 처리 결과
INFO 17484 --- [nio-8080-exec-2] c.c.c.SpringMvcMainCoffeeController      : # call Spring MVC Main Controller: xxxx-xx-xxT14:xx:xx.xxxxxxxxx // (2-1) 두 번째 요청
INFO 17484 --- [           main] c.c.SpringMvcMainSampleApplication       : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼 // (2-2) 두 번째 요청 처리 결과
INFO 17484 --- [nio-8080-exec-3] c.c.c.SpringMvcMainCoffeeController      : # call Spring MVC Main Controller: xxxx-xx-xxT14:xx:xx.xxxxxxxxx
INFO 17484 --- [           main] c.c.SpringMvcMainSampleApplication       : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
INFO 17484 --- [nio-8080-exec-4] c.c.c.SpringMvcMainCoffeeController      : # call Spring MVC Main Controller: xxxx-xx-xxT14:xx:xx.xxxxxxxxx
INFO 17484 --- [           main] c.c.SpringMvcMainSampleApplication       : xx:xx:xx.xxxxxx: coffee name: 카페라떼
INFO 17484 --- [nio-8080-exec-5] c.c.c.SpringMvcMainCoffeeController      : # call Spring MVC Main Controller: xxxx-xx-xxT14:xx:xx.xxxxxxxxx
INFO 17484 --- [           main] c.c.SpringMvcMainSampleApplication       : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
```

- 메인 애플리케이션 Controller(SpringMvcMainCoffeeController)의 getCoffee() 핸들러 메서드를 호출하는 클라이언트를 구현한 코드
- (1)에서 for 문을 이용해서 SpringMvcMainCoffeeController의 getCoffee() 핸들러 메서드(http://localhost:8080/coffees/1)를 다섯 번 호출함
    - 애플리케이션을 실행할 때, 메인 애플리케이션이 호출하는 대상인 외부 애플리케이션을 먼저 실행시킨 후에 메인 애플리케이션을 실행해야 함
- Spring에서는 CommandLineRunner나 ApplicationRunner를 이용하면 애플리케이션이 실행되는 시점에 어떤 처리 작업을 할 수 있음
    - CommandLineRunner 내부의 코드가 메인 애플리케이션 SpringMvcMainCoffeeController의 getCoffee() 핸들러 메서드에 요청을 전송하는 클라이언트의 역할을 함
- (1-1)에서 첫 번째 요청이 SpringMvcMainCoffeeController의 getCoffee() 핸들러 메서드에 전달된 시간과 (1-2)에서 첫 번째 요청 처리 결과로 출력된 시간을 보면 5초 정도의 시간이 걸린 것을 확인할 수 있음
- 나머지 요청 처리 시간 역시 5초 정도 걸렸으며, 이처럼 각 요청마다 5초의 시간이 걸린 이유는 외부 애플리케이션 역할을 하는 SpringMvcOutboundCoffeeController의 getCoffee() 핸들러 메서드에서 Thread.sleep(5000)를 설정해서 요청 처리 스레드에 5초의 지연 시간을 주었기 때문으로, 5번의 요청에 걸린 시간은 총 25초 정도가 됨
- 이 결과를 통해 Spring MVC 기반의 메인 애플리케이션이 외부 애플리케이션 서버와 통신할 때 요청 처리 스레드가 Blocking 된다는 것을 알 수 있음

<br>

##### Spring WebFlux의 Non-Blocking 처리 방식

```java
// Spring WebFlux 기반 메인 애플리케이션의 Controller 샘플 코드
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

import java.time.LocalDateTime;

@Slf4j
@RestController
@RequestMapping("/coffees")
public class SpringWebFluxMainCoffeeController {
    String uri = "http://localhost:5050/coffees/1";

    @ResponseStatus(HttpStatus.OK)
    @GetMapping("/{coffee-id}")
    public Mono<CoffeeResponseDto> getCoffee(@PathVariable("coffee-id") long coffeeId) throws InterruptedException {
        log.info("# call Spring WebFlux Main Controller: {}", LocalDateTime.now());

        // (1)
        return WebClient.create()
                .get()
                .uri(uri)
                .retrieve()
                .bodyToMono(CoffeeResponseDto.class);
    }
}
```

- Spring WebFlux 기반의 메인 애플리케이션 역시 클라이언트의 요청을 처리하기 위해 (1)과 같이 외부 애플리케이션에 한번 더 요청을 전송함
- Spring MVC 기반의 애플리케이션 Controller 코드와 두 가지 차이점이 존재함
    - 첫 번째, Spring MVC 기반 애플리케이션에서는 외부 애플리케이션과의 통신을 위해 RestTemplate을 사용한 반면 Spring WebFlux 기반 애플리케이션에서는 (1)과 같이 WebClient라는 Rest Client를 사용함
        - RestTemplate이 Blocking 방식의 Rest Client인 반면에 WebClient는 Non-Blocking 방식의 Rest Client
    - 두 번째, getCoffee() 핸들러 메서드의 리턴 타입이 ResponseEntity&#60;CoffeeResponseDto&#62;가 아니라 Mono&#60;CoffeeResponseDto&#62;라는 것

<br>

```java
// Spring WebFlux 기반 외부 애플리케이션의 Controller 샘플 코드
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/coffees")
public class SpringWebFluxOutboundCoffeeController {
    @ResponseStatus(HttpStatus.OK)
    @GetMapping("/{coffee-id}")
    public Mono<CoffeeResponseDto> getCoffee(@PathVariable("coffee-id") long coffeeId) throws InterruptedException {
        CoffeeResponseDto responseDto = new CoffeeResponseDto(coffeeId, "카페라떼", "CafeLattee", 4000);

        // (1)
        Thread.sleep(5000);
        return Mono.just(responseDto);   // (2)
    }
}
```

- 메인 애플리케이션 Controller(SpringWebFluxMainCoffeeController)의 getCoffee() 핸들러 메서드에서 WebClient로 전송한 요청을 전달받는 외부 애플리케이션의 Controller(SpringWebFluxOutboundCoffeeController) 코드
- 외부 애플리케이션의 요청 처리 시간이 오래 걸리는 것을 시뮬레이션하기 위해서 (1)과 같이 요청 처리 스레드에 지연 시간을 주었음

<br>

```java
import com.CoffeeResponseDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

import java.time.LocalTime;

@Slf4j
@SpringBootApplication
public class SpringWebFluxMainSampleApplication {

	public static void main(String[] args) {
		System.setProperty("reactor.netty.ioWorkerCount", "1");
		SpringApplication.run(SpringWebFluxMainSampleApplication.class, args);
	}

	@Bean
	public CommandLineRunner run() {
		return (String... args) -> {
			log.info("# 요청 시작 시간: {}", LocalTime.now());

      // (1)
			for (int i = 1; i <= 5; i++) {
				this.getCoffee()
						.subscribe(
								response -> {
									log.info("{}: coffee name: {}", LocalTime.now(), response.getKorName());
								}
						);
			}
		};
	}

	private Mono<CoffeeResponseDto> getCoffee() {
		String uri = "http://localhost:6060/coffees/1";

		return WebClient.create()
				.get()
				.uri(uri)
				.retrieve()
				.bodyToMono(CoffeeResponseDto.class);
	}
}

// 실행 결과
INFO 20160 --- [           main] c.c.SpringWebFluxMainSampleApplication   : # 요청 시작 시간: xx:xx:xx.xxxxxxxxx
INFO 20160 --- [ctor-http-nio-1] c.c.c.SpringWebFluxMainCoffeeController  : # call Spring WebFlux Main Controller: xxxx-xx-xxT15:xx:xx.xxxxxxxxx // (1) 첫 번째 요청 수신
INFO 20160 --- [ctor-http-nio-1] c.c.c.SpringWebFluxMainCoffeeController  : # call Spring WebFlux Main Controller: xxxx-xx-xxT15:xx:xx.xxxxxxxxx // (2) 두 번째 요청 수신
INFO 20160 --- [ctor-http-nio-1] c.c.c.SpringWebFluxMainCoffeeController  : # call Spring WebFlux Main Controller: xxxx-xx-xxT15:xx:xx.xxxxxxxxx // (3) 세 번째 요청 수신
INFO 20160 --- [ctor-http-nio-1] c.c.c.SpringWebFluxMainCoffeeController  : # call Spring WebFlux Main Controller: xxxx-xx-xxT15:xx:xx.xxxxxxxxx // (4) 네 번째 요청 수신
INFO 20160 --- [ctor-http-nio-1] c.c.c.SpringWebFluxMainCoffeeController  : # call Spring WebFlux Main Controller: xxxx-xx-xxT15:xx:xx.xxxxxx    // (5) 다섯 번째 요청 수신
INFO 20160 --- [ctor-http-nio-1] c.c.SpringWebFluxMainSampleApplication   : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
INFO 20160 --- [ctor-http-nio-1] c.c.SpringWebFluxMainSampleApplication   : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
INFO 20160 --- [ctor-http-nio-1] c.c.SpringWebFluxMainSampleApplication   : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
INFO 20160 --- [ctor-http-nio-1] c.c.SpringWebFluxMainSampleApplication   : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
INFO 20160 --- [ctor-http-nio-1] c.c.SpringWebFluxMainSampleApplication   : xx:xx:xx.xxxxxxxxx: coffee name: 카페라떼
```

- Spring WebFlux 기반의 메인 애플리케이션 Controller(SpringWebFluxMainCoffeeController)의 getCoffee() 핸들러 메서드를 호출하는 클라이언트를 구현한 코드
- (1)과 같이 for 문을 이용해서 SpringWebFluxMainCoffeeController의 getCoffee() 핸들러 메서드(http://localhost:6060/coffees/1)를 다섯 번 호출함
- 요청 시작 시간과 요청 결과로 전달받은 커피 정보를 수신한 시간을 확인해 보면 전체 처리 시간이 6초 정도 소요됨
- (1)부터 (5)까지의 요청 수신 시간이 밀리초 단위는 조금 다르지만 초 단위는 동일함
    - 이 결과의 의미는 메인 애플리케이션의 Controller에서 외부 애플리케이션의 Controller에 요청을 전송할 때, 외부 애플리케이션의 Controller에서 Thread.sleep(5000)으로 지연 시간을 주었다고 해서 메인 애플리케이션 Controller의 요청 처리 스레드가 Blocking 되지 않는다는 것을 의미함
    - 거의 동일한 시간에 동시 다발적으로 요청이 들어와도 요청을 일단 Blocking 하지 않고, 외부 애플리케이션에게 요청을 그대로 전달함
    - 이렇게 하면 외부 애플리케이션 Controller에서 5초 정도의 지연시간을 주긴 했지만 한꺼번에 다섯 개의 요청을 거의 동시에 외부 애플리케이션 Controller에 전송하는 효과를 보이기 때문에 전체 처리 시간 역시 5초를 조금 넘는 6초의 시간이 걸리는 것
- 1차로 요청을 수신한 애플리케이션에서 외부 애플리케이션에 요청을 추가적으로 전달할 때 1차로 요청을 수신한 애플리케이션의 요청 처리 스레드가 Blocking 되지 않음