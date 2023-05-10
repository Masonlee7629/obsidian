태그: #Spring
연결문서: [Spring MVC - API 계층 3](Spring%20MVC%20-%20API%20계층%203.md)

# Controller

## HTTP 헤더(Header)

HTTP 헤더는 HTTP 메시지(Messages)의 구성 요소 중 하나로써 클라이언트의 요청이나 서버의 응답에 포함되어 부가적인 정보를 HTTP 메시지에 포함할 수 있도록 해준다.

### HTTP 헤더의 사용 목적

#### 클라이언트와 서버 관점에서의 대표적인 HTTP 헤더 예시

클라이언트와 서버의 관점에서 내부적으로 가장 많이 사용되는 헤더 정보로 'Content-Type'이 있다.  
  

'Content-Type' 헤더 정보는 클라이언트와 서버가 주고 받는 HTTP 메시지 바디의 데이터 형식이 무엇인지를 알려주는 역할을 한다.  
  

클라이언트와 서버는 이 'Content-type'이 명시된 데이터 형식에 맞는 데이터들을 주고 받는 것이다.

#### 개발자들이 실무에서 사용하는 대표적인 HTTP 헤더 예시

-   **Authorization**  
    ‘Authorization’ 헤더 정보는 클라이언트가 적절한 자격 증명을 가지고 있는지를 확인하기 위한 정보이다.  
      
    일반적으로 REST API 기반 애플리케이션의 경우 클라이언트와 서버 간의 로그인 인증에 통과한 클라이언트들은 'Authorization' 헤더 정보를 기준으로 인증에 통과한 클라이언트가 맞는지 확인하는 절차를 거친다.
-   **User-Agent**  
    실무에서 애플리케이션을 구현하다보면 여러 유형의 클라이언트가 하나의 서버 애플리케이션에 요청을 전송하는 경우가 많다.  
      
    그리고 데스크탑과 모바일에서 들어오는 요청을 구분해서 응답 데이터를 다르게 보내줘야 하는 경우가 있을 수 있다.  
      
    이 경우, 'User-Agent' 정보를 이용하여 모바일 에이전트에서 들어오는 모청인지 모바일 이외에 다른 에이전트에서 들어오는 요청인지를 구분해서 처리할 수 있다.

### HTTP Request 헤더 정보 얻기

Spring MVC는 HTTP 헤더 정보를 읽어오는 몇 가지 방법을 제공하고 있다.

-   @RequestHeader로 개별 헤더 정보 받기

```
@RestController
@RequestMapping(path = "/v1/members")
public class MemberController {
    @PostMapping
    public ResponseEntity postMember(@RequestHeader("user-agent") String userAgent,//(1)
                                     @RequestParam("korName") String korName,
                                     @RequestParam("engName") String engName) {
        System.out.println("user-agent: " + userAgent);
        return new ResponseEntity<>(new Member(korName, engName),
                HttpStatus.CREATED);
    }
}
```

-   @RequestHeader로 전체 헤더 정보 받기

```
@RestController
@RequestMapping(path = "/v1/members")
public class MemberController {
    @PostMapping
    public ResponseEntity postMember(@RequestHeader Map<String, String> headers,//(1)
                                     @RequestParam("email") String email,
                                     @RequestParam("name") String name,
                                     @RequestParam("phone") String phone) {
        for (Map.Entry<String, String> entry : headers.entrySet()) {
            System.out.println("key: " + entry.getKey() +
                    ", value: " + entry.getValue());
        }

        return new ResponseEntity<>(new Member(email, name, phone),
                HttpStatus.CREATED);
    }
}
```

-   HttpServletRequest 객체로 헤더 정보 얻기

```
@RestController
@RequestMapping(path = "/v1/orders")
public class OrderController {
    @PostMapping
    public ResponseEntity postOrder(HttpServletRequest httpServletRequest,//(1)
                                    @RequestParam("memberId") long memberId,
                                    @RequestParam("coffeeId") long coffeeId) {
        System.out.println("user-agent: " + httpServletRequest.getHeader("user-agent"));

        return new ResponseEntity<>(new Order(memberId, coffeeId),
                HttpStatus.CREATED);
    }
}
```

-   HttpEntity 객체로 헤더 정보 얻기

```
@RestController
@RequestMapping(path = "/v1/coffees")
public class CoffeeController{
    @PostMapping
    public ResponseEntity postCoffee(@RequestHeader("user-agent") String userAgent,//(1)
                                     @RequestParam("korName") String korName,
                                     @RequestParam("engName") String engName,
                                     @RequestParam("price") int price) {
        System.out.println("user-agent: " + userAgent);
        return new ResponseEntity<>(new Coffee(korName, engName, price),
                HttpStatus.CREATED);
    }

    @GetMapping
    public ResponseEntity getCoffees(HttpEntity httpEntity) {
        for(Map.Entry<String, List<String>> entry : httpEntity.getHeaders().entrySet()){
            System.out.println("key: " + entry.getKey()
                    + ", " + "value: " + entry.getValue());
        }

        System.out.println("host: " + httpEntity.getHeaders().getHost());
        return null;
    }
}
```

### HTTP Response 헤더 정보 추가

클라이언트가 전달하는 Response에 헤더 정보를 추가하는 방법이다.

-   ResponseEntity와 HttpHeaders를 이용해 헤더 정보 추가하기

```
@RestController
@RequestMapping(path = "/members")
public class MemberController{
    @PostMapping
    public ResponseEntity postMember(@RequestParam("email") String email,
                                     @RequestParam("name") String name,
                                     @RequestParam("phone") String phone) {
        // (1) 위치 정보를 헤더에 추가
        HttpHeaders headers = new HttpHeaders();
        headers.set("Client-Geo-Location", "Korea,Seoul");

        return new ResponseEntity<>(new Member(email, name, phone), headers,
                HttpStatus.CREATED);
    }
}
```

-   HttpServletResponse 객체로 헤더 정보 추가하기

```
@RestController
@RequestMapping(path = "/v1/members")
public class MemberController{
    @GetMapping
    public ResponseEntity getMembers(HttpServletResponse response) {
        response.addHeader("Client-Geo-Location", "Korea,Seoul");

        return null;
    }
}
```

---

## Rest Client

### 클라이언트(Client)와 서버(Server)의 관계

> 어떤 서버가 HTTP 통신을 통해서 다른 서버의 리소스를 이용한다면 그 때만큼은 클라이언트의 역할을 한다.  
>   

웹 브라우저는 웹 브라우저에서 보여지는 HTML 컨텐츠를 웹 서버에 요청하고, 웹 서버는 요청에 해당하는 적절한 컨텐츠를 웹 브라우저에 응답으로 전달한다.  
  

즉, 웹 브라우저는 웹 서버로부터 HTML 컨텐츠를 제공받는 클라이언트가 된다.  
  

웹 브라우저는 웹 서버가 응답으로 전달해주는 HTML 컨텐츠를 전달 받아서 브라우저 내에 보여준다.  
  

여기서 서버 쪽의 **리소스를 이용하는 측**이 클라이언트가 되는 것이다.  
  

그런데 서버는 항상 클라이언트에게 리소스를 제공하는 역할만 하는 것이 아니라 **서버도 다른 서버로부터 리소스를 제공 받아야 하는 경우가 굉장히 많다.**  
  

대표적인 예가 Frontend 서버와 Backend 서버의 관계이다.

### Rest Client

Rest Client란 Rest API 서버에 HTTP 요청을 부낼 수 있는 클라이언트 툴 또는 라이브러리를 의미한다.  
  

예시로 Postman은 UI가 갖춰진 Rest Client라고 할 수 있다.

### RestTemplate

Java에서 사용할 수 있는 HTTP Client 라이브러리로는 java.net.HttpURLConnection, Apache HttpComponents, OkHttp 3, Netty 등이 있다.  
  

Spring에서는 이 HTTP Client 라이브러리 중 하나를 이용해서 원격지에 있는 다른 Backend 서버에 HTTP 요청을 보낼 수 있는 RestTemplate이라는 꽤 괜찮은 Rest Client API를 제공한다.  
  

RestTemplate을 이용하면 Rest 엔드 포인트 지정, 헤더 설정, 파라미터 및 body 설정을 한 줄의 코드로 손쉽게 전송할 수 있다.

#### RestTemplate 이용 과정

-   **RestTemplate 객체 생성**  
    RestTemplate의 객체를 생성하기 위해서는 RestTemplate의 생성자 파라미터로 HTTP Client 라이브러리의 구현 객체를 전달해야 한다.

예시에선 HttpComponentsClientHttpRequestFactory 클래스를 통해 Apache HttpComponents를 전달한다.

```
public class RestClientExample01 {
    public static void main(String[] args) {
        // (1) 객체 생성
        RestTemplate restTemplate = 
                new RestTemplate(new HttpComponentsClientHttpRequestFactory());
    }
}
```

```
// build.gradle 의 dependecies 항목에 의존 라이브러리 추가
dependencies {
    ...
    ...
    implementation 'org.apache.httpcomponents:httpclient'
}
```

-   **URI 생성**  
    RestTemplate 객체를 생성한 후, HTTP Request를 전송할 Rest 엔드포인트의 URI를 지정해야한다.  
      
    예시에선 UriComponentsBuilder 클래스를 이용해서 UriComponents 객체를 생성한 후에 이 UriComponents 객체를 이용해서 HTTP Request를 요청할 엔드포인트의 URI를 생성하고 있다.

**UriComponentsBuilder 클래스에서 제공하는 API 메서드의 기능**

-   -   newInstance()
        -   UriComponentsBuilder 객체를 생성한다.
    -   scheme()
        -   URI의 scheme을 설정한다.
    -   host()
        -   호스트 정보를 입력한다.
    -   port()
        -   디폴트 값은 80 이므로 80 포트를 사용하는 호스트라면 생략 가능하다.
    -   path()
        -   URI의 경로(path)를 입력한다.
        -   예시에서는 URI의 path에서 {continents}, {city} 의 두 개의 템플릿 변수를 사용하고 있다.
        -   두 개의 템플릿 변수는 uriComponents.expand("Asia", "Seoul").toUri(); 에서 expand() 메서드 파라미터의 문자열로 채워진다.
        -   즉, 빌드 타임에 {continents}는 ‘Asia’, {city}는 ‘Seoul’로 변환된다.
    -   encode()
        -   URI에 사용된 템플릿 변수들을 인코딩 해준다.
        -   여기서 인코딩의 의미는 non-ASCII 문자와 URI에 적절하지 않은 문자를 Percent Encoding 한다는 의미이다.
    -   build()
        -   UriComponents 객체를 생성한다.
-   _UriComponents에 사용된 API 메서드의 기능\*_
    -   expand()
        -   파라미터로 입력한 값을 URI 템플릿 변수의 값으로 대체한다.
    -   toUri()
        -   URI 객체를 생성한다.

```
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponents;
import org.springframework.web.util.UriComponentsBuilder;

import java.net.URI;

public class RestClientExample01 {
    public static void main(String[] args) {
        // (1) 객체 생성
        RestTemplate restTemplate =
                new RestTemplate(new HttpComponentsClientHttpRequestFactory());

        // (2) URI 생성
        UriComponents uriComponents =
                UriComponentsBuilder
                        .newInstance()
                        .scheme("http")
                        .host("worldtimeapi.org")
//                        .port(80)
                        .path("/api/timezone/{continents}/{city}")
                        .encode()
                        .build();
        URI uri = uriComponents.expand("Asia", "Seoul").toUri();
    }
}
```

-   **요청 전송**  
    URI 설정까지 끝났다면 이제 엔드포인트로 Request를 전송한다.

> **getForObject(URI uri, Class<T> responseType)**
> -   기능
>     -   getForObject() 메서드는 HTTP Get 요청을 통해 서버의 리소스를 조회한다.
> -   파라미터
>     -   URI uri
>         -   Request를 전송할 엔드포인트의 URI 객체를 지정해준다.
>     -   Class<T> responseType
>         -   응답으로 전달 받을 클래스의 타입을 지정해준다.
>         -   예시에선 응답 데이터를 문자열로 받을 수 있도록 String.calss로 지정했다.
> 
> **exchange(URI uri, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType)**
> 
> -   기능
>     -   getForObject(), getForEntity() 등과 달리 exchange() 메서드는 HTTP Method, RequestEntity, ResponseEntity를 직접 지정해서 HTTP Request를 전송할 수 있는 가장 일반적인 방식이다.
> -   파라미터
>     -   URI url
>         -   Request를 전송할 엔드포인트의 URI 객체를 지정해준다.
>     -   HttpMethod method
>         -   HTTP Method 타입을 지정해준다.
>     -   HttpEntity<?> requestEntity
>         -   HttpEntity 객체를 지정해준다.
>         -   HttpEntity 객체를 통해 헤더 및 바디, 파라미터 등을 설정해줄 수 있다.
>     -   Class<T> responseType
>         -   응답으로 전달 받을 클래스의 타입을 지정해준다.

```
// getForObject()를 이용한 문자열 응답 데이터 전달 받기
public class RestClientExample01 {
    public static void main(String[] args) {
        // (1) 객체 생성
        RestTemplate restTemplate =
                new RestTemplate(new HttpComponentsClientHttpRequestFactory());

        // (2) URI 생성
        UriComponents uriComponents =
                UriComponentsBuilder
                        .newInstance()
                        .scheme("http")
                        .host("worldtimeapi.org")
//                        .port(80)
                        .path("/api/timezone/{continents}/{city}")
                        .encode()
                        .build();
        URI uri = uriComponents.expand("Asia", "Seoul").toUri();

        // (3) Request 전송
        String result = restTemplate.getForObject(uri, String.class);

        System.out.println(result);
    }
}
```

```
// getForObject()를 이용한 커스텀 클래스 타입으로 원하는 정보만 응답으로 전달 받기
// 전달 받고자하는 응답 데이터의 JSON 프로퍼티 이름과 클래스의 멤버 변수 이름이 동일해야하고
// 해당 멤버 변수에 접근하기 위한 getter 메서드 역시 동일한 이름이어야 한다.
public class RestClientExample02 {
    public static void main(String[] args) {
        // (1) 객체 생성
        RestTemplate restTemplate =
                new RestTemplate(new HttpComponentsClientHttpRequestFactory());

        // (2) URI 생성
        UriComponents uriComponents =
                UriComponentsBuilder
                        .newInstance()
                        .scheme("http")
                        .host("worldtimeapi.org")
//                        .port(80)
                        .path("/api/timezone/{continents}/{city}")
                        .encode()
                        .build();
        URI uri = uriComponents.expand("Asia", "Seoul").toUri();

        // (3) Request 전송. WorldTime 클래스로 응답 데이터를 전달 받는다.
        WorldTime worldTime = restTemplate.getForObject(uri, WorldTime.class);

        System.out.println("# datatime: " + worldTime.getDatetime());
        System.out.println("# timezone: " + worldTime.getTimezone());
        System.out.println("# day_of_week: " + worldTime.getDay_of_week());
    }
}

---

public class WorldTime {
    private String datetime;
    private String timezone;
    private int day_of_week;

    public String getDatetime() {
        return datetime;
    }

    public String getTimezone() {
        return timezone;
    }

    public int getDay_of_week() {
        return day_of_week;
    }
}
```

```
// getForEntity()를 사용한 Response Body(바디, 컨텐츠) + Header(헤더) 정보 전달 받기
public class RestClientExample02 {
    public static void main(String[] args) {
        // (1) 객체 생성
        RestTemplate restTemplate =
                new RestTemplate(new HttpComponentsClientHttpRequestFactory());

        // (2) URI 생성
        UriComponents uriComponents =
                UriComponentsBuilder
                        .newInstance()
                        .scheme("http")
                        .host("worldtimeapi.org")
//                        .port(80)
                        .path("/api/timezone/{continents}/{city}")
                        .encode()
                        .build();
        URI uri = uriComponents.expand("Asia", "Seoul").toUri();

        // (3) Request 전송. ResponseEntity로 헤더와 바디 정보를 모두 전달 받을 수 있다.
        ResponseEntity<WorldTime> response =
                restTemplate.getForEntity(uri, WorldTime.class);

        System.out.println("# datatime: " + response.getBody().getDatetime());
        System.out.println("# timezone: " + response.getBody().getTimezone()());
        System.out.println("# day_of_week: " + response.getBody().getDay_of_week());
        System.out.println("# HTTP Status Code: " + response.getStatusCode());
        System.out.println("# HTTP Status Value: " + response.getStatusCodeValue());
        System.out.println("# Content Type: " + response.getHeaders().getContentType());
        System.out.println(response.getHeaders().entrySet());
    }
}
```

```
// exchange() 를 사용한 응답 데이터 받기
// exchange() 메서드를 사용한 방식은 이전 예시의 방식들보다 조금 더 일반적인 HTTP Request 방식으로
// HTTP Method나 HTTP Request, HTTP Response 방식을 개발자가 직접 지정해서 유연하게 사용할 수 있다.
public class RestClientExample03 {
    public static void main(String[] args) {
        // (1) 객체 생성
        RestTemplate restTemplate =
                new RestTemplate(new HttpComponentsClientHttpRequestFactory());

        // (2) URI 생성
        UriComponents uriComponents =
                UriComponentsBuilder
                        .newInstance()
                        .scheme("http")
                        .host("worldtimeapi.org")
//                        .port(80)
                        .path("/api/timezone/{continents}/{city}")
                        .encode()
                        .build();
        URI uri = uriComponents.expand("Asia", "Seoul").toUri();

        // (3) Request 전송. exchange()를 사용한 일반화 된 방식
        ResponseEntity<WorldTime> response =
                restTemplate.exchange(uri,
                        HttpMethod.GET,
                        null,
                        WorldTime.class);

        System.out.println("# datatime: " + response.getBody().getDatetime());
        System.out.println("# timezone: " + response.getBody().getTimezone());
        System.out.println("# day_of_week: " + response.getBody().getDay_of_week());
        System.out.println("# HTTP Status Code: " + response.getStatusCode());
        System.out.println("# HTTP Status Value: " + response.getStatusCodeValue());
    }
}
```
