태그: #Cloud 

# 웹 서버

## TOMCAT

### Tomcat이란?
- Apache사에서 개발한 서블릿 컨테이너만 있는 오픈소스 웹 애플리케이션 서버
- Spring Boot의 내장 서버이기 때문에 별도의 설치 과정 없이 자연스럽게 Tomcat을 사용함
- Tomcat의 특징
    - 자바 애플리케이션을 위한 대표적인 오픈소스 WAS(Web Application Server)
    - 오픈소스이기 때문에 라이선스 비용 부담 없이 사용할 수 있음
    - 독립적으로도 사용 가능하며 Apache 같은 다른 웹 서버와 연동하여 함께 사용할 수 있음
    - Tomcat은 자바 서블릿 컨테이너에 대한 공식 구현체로, Spring Boot에 내장되어 있어 별도의 설치 과정이 필요하지 않음

<br>

### Tomcat 실행 및 의존성 확인하기
- IntelliJ 상단 메뉴바의 View > Tool Windows > Gradle을 클릭함
- Gradle 탭에선 프로젝트에서 추가한 의존성 모듈을 모두 확인할 수 있음
- spring-boot-starter-web 모듈의 토글을 열어보면, 해당 모듈이 포함하고 있는 다른 모듈을 확인할 수 있음
- 웹 서버를 구성하는 모듈 속엔 spring-boot-starter-tomcat 모듈도 포함하고 있음

<br>

---

<br>

## Jetty

### Jetty란?
- 이클립스 재단의 HTTP 서버이자 자바 서블릿 컨테이너
- Jetty도 Tomcat과 같이 자바 서블릿 컨테이너이자 서버로 사용할 수 있기 때문에 개발자는 원하는 서버를 선택하여 프로젝트를 구성할 수 있음
- Jetty의 특징
    - 2009년 이클립스 재단으로 이전하며 오픈소스 프로젝트로 개발됨
    - Jetty는 타 웹 애플리케이션 대비 적은 메모리를 사용하여 가볍고 빠름
    - 애플리케이션에 내장 가능함
    - 경량 웹 애플리케이션으로 소형 장비, 소규모 프로그램에 더 적합함

<br>

 ### Spring Boot 서버 Jetty로 변경하기
 - 아무런 설정을 해주지 않는다면 Spring Boot의 기본 내장 서버인 Tomcat으로 실행됨
 - build.gradle 파일에서 spring-boot-starter-web 의존성이 추가되어 있는 부분을 확인하여, 이 의존성 모듈 내에 포함되어 있는 Tomcat을 제외시킴
    - 제외시킨 후 프로젝트를 재 빌드하면 의존성이 제거되었음을 확인할 수 있음
<br>

- Tomcat을 제외했다면 대체할 서버로 Jetty 의존성을 추가함
    - 추가한 후 프로젝트를 재 빌드하면 Jetty에 대한 의존성이 추가되었음을 확인할 수 있음
<br>

```java
implementation ('org.springframework.boot:spring-boot-starter-web') {
		exclude module: 'spring-boot-starter-tomcat'
	}
implementation ('org.springframework.boot:spring-boot-starter-jetty')
```

<br>

---

<br>

## NginX - Proxy Server

###  NginX란?
- Nginx는 가볍고 높은 성능을 보이는 오픈소스 웹 서버 소프트웨어
- Tomcat과 Jetty는 자바 서블릿 컨테이너 혹은 웹 애플리케이션 서버였다면, Nginx는 웹 서버로 클라이언트에게 정적 리소스를 빠르게 응답하기 위한 웹 서버로 사용할 수 있음
- Nginx의 특징
    - Nginx는 트래픽이 많은 웹 사이트의 확장성을 위해 개발된 고성능 웹 서버
    - 비동기 이벤트 기반으로 적은 자원으로 높은 성능과, 높은 동시성을 위해 개발됨
    - 다수의 클라이언트 연결을 효율적으로 처리할 수 있음
    - 클라이언트와 서버 사이에 존재하는 리버스 프록시 서버로 사용할 수 있음
    - Nginx를 클라이언트와 서버 사이에 배치하여 무중단 배포를 할 수 있음

<br>

### Spring Boot와 Nginx 연동하기
- 지금까지 클라이언트는 8080번 포트를 사용하는 스프링부트로 요청을 보내고 응답을 받았으나, 이 구조에서 스프링부트 앞에 Nginx로 리버스 프록시 서버를 구축하여, 클라이언트와 서버가 Nginx를 통해서 소통할 수 있도록 함

<br>

#### 1. NginX 설치
- Mac OS 사용자
    - 터미널에 명령어를 입력함
    - 설치 진행이 되지 않는다면 brew update를 진행함
<br>

```bash
brew install nginx
```

<br>

- Windows 사용자
    - Nginx [다운로드 사이트](https://nginx.org/en/download.html)에서 Stable 버전을 다운로드함
    - 다운로드한 압축 폴더를 원하는 경로로 이동한 후 압축을 해제하며, 폴더 내에 Nginx의 실행파일과 Nginx 설정 폴더가 있으니 경로를 기억해야 함

<br>

#### 2. NginX 실행
- Mac OS
    - 설치가 완료되었다면 다음 명령어를 통해 Nginx를 실행함
<br>

```bash
brew services start nginx
```

<br>

- Windows
    - 압축을 해제한 폴더의 nginx.exe를 실행함
    - 실행 시 방화벽 창이 뜬다면 추가 정보 버튼을 클릭해 실행버튼을 활성화함

<br>

##### 실행 확인
- 실행 후 localhost에 접근하여 연결이 잘 되는지 확인함
- MacOS는 8080번 포트(localhost:8080), Windows는 80번 포트(localhost:80)가 기본 포트로 설정되어 있음

<br>

#### 3. NginX 설정 파일 수정
- 연결이 원활하게 되었다면 Nginx의 설정파일을 수정해야 하며, 설정파일 수정 내용은 모든 OS 동일함
<br>

- Mac OS
    - 설정파일은 brew 버전에 따라 위치가 다를 수 있음
    - Nginx의 설정파일 위치는 아래 명령어로 확인할 수 있음
    - nano 에디터를 이용해 설정파일을 수정함
<br>

```bash
$ nginx -t
$ nano /opt/homebrew/etc/nginx/nginx.conf
```

<br>

- Windows
    - Nginx 실행파일이 있던 폴더(압축 해제한 폴더)에 설정파일이 모여있는 conf 폴더가 있음
    - conf 폴더 내의 nginx.conf 파일을 수정하며, 우클릭 후 IntelliJ에서 수정하거나 터미널에서 nano 에디터로 수정할 수 있음

<br>

##### 설정 파일 수정
- 설정파일에는 Nginx의 다양한 설정값이 있음
- 설정에서 포트 번호를 80으로 변경하고 리버스 프록시 서버로서 Spring Boot 프로젝트에 연동함
<br>

```bash
server {
		listen       80; # (Mac OS) 8080 포트에서 80번 포트로 변경함
...
		location / {
				...
				proxy_pass http://localhost:8080; # 요청을 8080 포트로 넘김
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header Host $http_host;
		}
}
```

<br>

#### 4. NginX 재시작
- 설정파일을 저장한 후 Nginx를 재시작함
<br>

- Mac OS
<br>

```bash
$ brew services restart nginx
```

<br>

- Windows
    - 명령 프롬프트를 실행함
    - Nginx 실행 파일이 있는 폴더(압축 해제한 폴더)로 이동하여 다음 명령어를 실행함
<br>

```bash
$ nginx -s reload
```

<br>

- Nginx 재시작 후 localhost (80번 포트는 생략 가능)에 접속하면, 80번 포트로 수정한 Nginx에는 접속이 되지만, 아직 8080번 포트를 사용하지 않기 때문에 오류가 발생함
- Spring Boot(8080번 포트)를 실행해야 하며, Spring Boot가 8080번 포트에서 정상적으로 실행이 되었다면 브라우저를 새로고침함

<br>

#### NginX 종료
- Mac OS
<br>

```bash
$ brew services stop nginx
```

<br>

- Windows 명령 프롬프트
<br>

```bash
$ nginx -s stop
```

<br>

---

<br>

## NginX - Load Balancer
- NGINX를 이용하여 로컬 환경에서 로드밸런싱을 구성함
- 클라이언트 - NGINX - Spring Boot로 이전의 프록시 서버와 비슷한 구조
- NGINX는 스프링부트 서버 앞에 위치하여 클라이언트와 서버가 NGINX를 통해 소통하도록 되어있음

<br>

### 1. 두 개의 스프링부트 서버 실행
- NGINX를 이용한 로드밸런싱을 구현하기 전, 두 개의 스프링부트 서버를 각각 실행함
- 하나는 8080번 포트, 다른 하나는 8081번 포트를 사용해서 실행함
    - 포트 번호는 동일하지 않아도 상관 없음
<br>

- 프로젝트를 빌드함
<br>

```bash
./gradlew build
```

<br>

- 빌드 파일을 실행함 (포트를 변경하지 않은 경우 8080번 포트에서 실행됨)
<br>

```bash
java -jar sample-0.0.1-SNAPSHOT.jar
```

<br>

- 빌드 파일을 이용하여 새 터미널에 다른 포트(8081번 포트)에서 실행함
<br>

```bash
java -Dserver.port=8081 -jar sample-0.0.1-SNAPSHOT.jar
```

<br>

- 8080번 포트로 실행한 프로젝트의 PID와 8081번 포트로 실행한 프로젝트의 PID가 서로 다른 PID를 통해 실행됨
- 두 프로젝트는 서로 별개의 프로세스를 가지며 각각 실행되며, 어느 하나를 멈춘다고 해도 다른 포트로 열린 프로젝트에 영향이 가지 않음

<br>

### 2. NginX 설정파일 수정
- NGINX 설정파일을 수정함
<br>

```java
http {
	upstream backend {
		server localhost:8080;
		server localhost:8081;
	}
	location / {
		proxy_pass http://backend;
	}
}
```

<br>

- backend라는 서버 그룹을 만든 뒤 그룹 자체로 전달을 하는 구조
    - 서버 그룹의 이름은 다른 이름으로 변경 가능함
- 그룹에 속한 각 서버의 값은 실행한 두 개의 스프링부트 프로젝트 접속 URL을 작성함
    - 로컬환경에서 8080번 포트와 8081번 포트로 실행했기 때문에 각각 포트 번호까지 함께 작성함
- 서버 그룹을 모두 작성했다면 location의 proxy_pass 값으로 해당 서버 그룹을 설정함
- NGINX의 포트는 80번으로 설정되어 있기 때문에 포트를 생략한 localhost로 접속 시 8080번 포트와 8081번 포트가 번갈아 연결됨

<br>

### 3. 로드밸런싱 결과
- NGINX를 실행하며, NGINX는 localhost(80번 포트 생략가능)로 실행됨
- 새로고침을 하면 두 프로세스가 번갈아 나오게 됨

<br>

---

<br>

# VPC

## VPC 관련 개념

### VPC
- VPC는 Virtual Private Cloud 서비스로, 클라우드 내 프라이빗 공간을 제공함으로써, 클라우드를 퍼블릭과 프라이빗 영역으로 논리적으로 분리할 수 있게 함
- 이전에 VPC가 없었을 때는 클라우드에 있는 리소스를 격리할 수 있는 방법이 없었고, 따라서 인스턴스들이 서로 거미줄처럼 연결되고, 인터넷과 연결되어 시스템의 복잡도를 엄청나게 끌어올릴 뿐만 아니라, 의존도를 높였음
    - 따라서 유지 및 관리에 많은 비용과 노력을 투입해야 하는 단점이 있었음
- VPC를 분리함으로써 확장성을 가질 수 있고, 네트워크에 대한 완전한 통제권을 가질 수 있음

<br>

###  VPC 이해를 위한 구성 요소와 주요 용어

#### IP Address
- IP는 컴퓨터 네트워크에서 장치들이 서로를 인식하고 통신을 하기 위해서 사용하는 특수한 번호로, IPv4, IPv6로 나뉘어 있으며 혼용하여 사용하고 있음
- IPv4를 기준으로 예를 들면, 아래와 같은 형식인 172.16.0.0의 모습으로 이루어져 있음
<br>

| &nbsp;&nbsp;172.&nbsp;&nbsp; | &nbsp;&nbsp;16.&nbsp;&nbsp; | &nbsp;&nbsp;0.&nbsp;&nbsp; | &nbsp;&nbsp;0&nbsp;&nbsp; |
| :-: | :-: | :-: | :-: |
| &nbsp;&nbsp;10101100.&nbsp;&nbsp; | &nbsp;&nbsp;00010000.&nbsp;&nbsp; | &nbsp;&nbsp;00000000.&nbsp;&nbsp; | &nbsp;&nbsp;00000000&nbsp;&nbsp; |

<br>

- 표에서 보이는 십진수의 형태는 보기 편하도록 변형한 것이고, 실제의 형태는 2진수 8자리의 형태, 즉 각 8bit(비트)씩 총 32bit로 구성되어 있음
- 8bit를 Octet(옥텟)이라고 부르며, '.' 으로 구분함
    - IPv4는 4개의 Octet(옥텟)으로 이루어져 있다고 할 수 있음

<br>

#### IP Address Class
- 이전에는 IPv4 주소에서 호스트가 연결되어 있는 특정 네트워크를 가리키는 8비트의 네트워크 영역(Network Address)과 해당 네트워크 내에서 호스트의 주소(Host Address)를 가리키는 나머지 영역을 구분하기 위해서 클래스(Class)를 사용했음
- 클래스는 총 5가지(A, B, C, D, E) 클래스로 나누어져 있음
    - D와 E 클래스는 멀티캐스트용, 연구 개발을 위한 예약 IP이므로 보통 사용되지 않음

<br>

- A 클래스
    - 0.0.0.0의 경우는 자체 네트워크를 의미하기 때문에 제외되고, 127.0.0.0 ~ 127.255.255.255는 자기 자신을 가리키기 위한 목적으로 예약된 IP주소이기 때문에 사용할 수 없음
<br>

| &nbsp;&nbsp;Network Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; |
| :-: | :-: | :-: | :-: |
| &nbsp;&nbsp;0-127&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; |

<br>

- B 클래스
<br>

| &nbsp;&nbsp;Network Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; |
|:---------------------------------------:|:------------------------------------:|:------------------------------------:|:------------------------------------:|
| &nbsp;&nbsp;128-191&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; |

<br>

- C 클래스
<br>

| &nbsp;&nbsp;Network Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; | &nbsp;&nbsp;Host Address&nbsp;&nbsp; |
|:---------------------------------------:|:------------------------------------:|:------------------------------------:|:------------------------------------:|
| &nbsp;&nbsp;192-223&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; | &nbsp;&nbsp;0-255&nbsp;&nbsp; |

<br>

### CIDR(Classless inter-domain routing)
- CIDR는 사이더라고 불리며, 클래스 없는 도메인 간 라우팅 기법으로 1993년 도입되기 시작한 국제 표준의 IP주소 할당 방법이며, 앞서 설명한 IP 클래스 방식을 대체한 방식
- 기존에는 클래스에 따라 정해진 Network Address와 Host Address를 사용해야 했다면, CIDR은 원하는 블록만큼 Network Address를 지정하여 운용할 수 있음

<br>

![](image_66.png)

<br>

- /16은 첫 16bit를 Network Address로 사용한다는 의미로, 총 2^16인 65,536개의 IP주소를 사용할 수 있는 커다란 네트워크 블록을 이러한 방식으로 표시함
<br>

| &nbsp;&nbsp;CIDR 블록&nbsp;&nbsp; | &nbsp;&nbsp;IP 주소의 수&nbsp;&nbsp; |
| :-: | :-: |
| &nbsp;&nbsp;/28&nbsp;&nbsp; | &nbsp;&nbsp;16&nbsp;&nbsp; |
| &nbsp;&nbsp;/24&nbsp;&nbsp; | &nbsp;&nbsp;254&nbsp;&nbsp; |
| &nbsp;&nbsp;/20&nbsp;&nbsp; | &nbsp;&nbsp;4,094&nbsp;&nbsp; |
| &nbsp;&nbsp;/18&nbsp;&nbsp; | &nbsp;&nbsp;16,382&nbsp;&nbsp; |
| &nbsp;&nbsp;/16&nbsp;&nbsp; | &nbsp;&nbsp;65,536&nbsp;&nbsp; |

<br>

### 서브넷(Subnet)
- 서브넷은 서브네트워크(Subnetwork)의 줄임말로 IP 네트워크의 논리적인 하위 부분을 가리킴
- 서브넷을 통해 하나의 네트워크를 여러 개로 나눌 수 있음
- VPC를 사용하면 퍼블릭 서브넷, 프라이빗 서브넷, VPN only 서브넷 등, 필요에 따라 다양한 서브넷을 생성할 수 있음
    - 퍼블릭 서브넷 : 인터넷을 통해 연결할 수 있는 서브넷
    - 프라이빗 서브넷 : 인터넷을 연결하지 않고, 보안을 유지하는 배타적인 서브넷
    - VPN only 서브넷 : 기업 데이터 센터와 VPC를 연결하는 서브넷
- 서브넷은 VPC의 CIDR 블록을 이용해 정의되며, 최소 크기의 서브넷은 /28
    - 이때 주의 할 점은 서브넷은 AZ당 최소 하나를 사용할 수 있고, 여러 개의 AZ에 연결되는 서브넷은 만들 수 없음
- AWS가 확보한 서브넷 중 처음 네 개의 IP주소와 마지막 IP주소는 인터넷 네트워킹을 위해 예약되어 있음
    - 서브넷에서 가용 IP주소를 계산할 때는 항상 이 부분을 기억하고 있어야 함
    - 예를 들어, 10.0.0.0/24 체계의 CIDR 블록이 있는 서브넷에서 10.0.0.0, 10.0.0.1, 10.0.02, 10.0.0.3, 10.0.0.255 등 5개의 IP주소는 예약되어 있음

<br>

### 라우팅 테이블(Routing Table)
- 라우팅 테이블은 트래픽의 전송 방향을 결정하는 라우트와 관련된 규칙을 담은 테이블로 목적지를 향한 최적의 경로로 데이터 패킷을 전송하기 위한 모든 정보를 담고 있음
- 라우팅 테이블은 하나의 지점에서 또 다른 지점으로 가기 위한 모든 정보를 제공하기 위한 테이블
- 모든 서브넷은 라우팅 테이블을 지님
- 특정 VPC의 서브넷이 라우팅 테이블에 인터넷 게이트웨이(VPC와 인터넷 간 통신을 가능하게 하는 구성요소)를 포함하고 있다면, 해당 서브넷은 인터넷 액세스 권한 및 정보를 가짐
- 각각의 서브넷은 항상 라우팅 테이블을 가지고 있어야 하며, 하나의 라우팅 테이블 규칙을 여러 개의 서브넷에 연결하는 것도 가능함
- 서브넷을 생성하고 별도의 라우팅 테이블을 생성하지 않으면 클라우드가 자동으로 VPC의 메인 라우팅 테이블을 연결함