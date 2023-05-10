태그: #Spring

# Spring MVC - 애플리케이션 빌드/실행/배포
## 애플리케이션 빌드
### IntelliJ IDE를 이용한 빌드
Spring Boot은 Gradle 빌드 툴을 이용해 애플리케이션을 빌드할 수 있는 플러그인을 지원하기 때문에 Gradle task 명령을 통해 애플리케이션을 손쉽게 빌드할 수 있다.
<br>

빌드 순서는 다음과 같다.
1. 우측 상단의 [Gradle] 윈도우 탭을 클릭한다.
2. 프로젝트 이름 > Tasks > build 에서 `:bootJar` 또는 `:build` task를 더블 클릭한다.

빌드가 정상적으로 종료되면 `build/libs` 디렉토리에 Jar 파일 하나가 생성되며, 해당 Jar 파일은 로컬 PC에서 실행 가능한 애플리케이션 실행 파일이다.
<br>

**`:build`와 `:bootJar`의 차이점**
`:build` task를 실행하면 `:assemble`, `:check` 같이 Gradle에서 빌드와 관련된 모든 task들을 실행시킵니다. 그리고 실행 가능한 Jar 파일 이외에 plain Jar 파일 하나를 더 생성한다.
반면에 `:bootJar`는 빌드와 관련된 모든 task들을 실행하는 것이 아니라 애플리케이션의 실행 가능한 Jar(Executable Jar) 파일을 생성하기 위한 task만 실행합니다. 따라서, 단순히 Executable Jar 파일만 필요하다면 `:bootJar` task를 실행하면 됩니다.

<br>

### Gradle Task를 이용한 빌드
IntelliJ IDE가 설치되어 있지 않은 상황에서 빌드를 해야 될 경우, Gradle task 명령어를 콘솔에서 바로 입력하여 빌드를 진행할 수 있다.
<br>

1. 프로젝트가 위치해 있는 디렉토리 경로로 이동한다.
<br>

2. Gradle task를 CLI 명령으로 입력할 수 있는 콘솔창을 프로젝트 root 경로에서 오픈한다.
<br>

3. 아래의 명령을 입력해서 애플리케이션 빌드를 진행한다.
- Windows 터미널의 경우
    
```
프로젝트_root_경로> .\gradlew bootJar
```

- Git Bash의 경우

```
$ ./gradlew build
```

<br>

### 애플리케이션 실행
빌드가 성공적으로 완료되었다면 이제 생성된 Jar(Executable Jar) 파일을 이용해서 애플리케이션을 실행할 수 있다.
애플리케이션 실행 순서는 다음과 같다.
1. 빌드를 통해 생성된 Jar 파일이 있는 디렉토리 경로로 이동한다.
2. 터미널 창을 오픈한 후, 다음과 같이 입력한다.
- `java -jar Jar_파일명.jar`
<br>

#### 프로파일(Profile) 적용
만약 빌드된 애플리케이션 실행 파일을 서버 환경에 배포해서 운영한다면 인메모리 DB를 사용하지 않아야 한다.
어떤 이유로 인해 서버가 다운된다거나 그로 인해 애플리케이션이 한 번이라도 재시작해야 되는 경우가 생긴다면 인메모리 DB에 저장된 데이터는 모두 초기화될 것이기 때문이다.
<br>

로컬 환경에서 개발을 진행할 때는 application.yml 파일에 H2를 사용하고, 서버용 jar 파일을 빌드할 경우에는 빌드 전에 기존에 application.yml 파일에 설정되어 있는 H2 정보 대신에 서버에서 사용하는 DB 정보로 수정한 뒤에 빌드하면 될 것이다.
다만, 매번 수정 작업을 진행하는 것은 비효율적이다.
<br>

Spring에서는 이런 불편함을 줄이기 위해 프로파일(Profile)이라는 아주 편리한 기능을 제공한다.

1. 기존의 application.yml 파일 외에 application-local.yml 파일과 application-server.yml 파일을 추가한다.
    - application.yml
        - application.yml 파일은 주로 애플리케이션의 실행 환경에 상관없이 공통적으로 적용할 수 있는 프로퍼티를 설정할 수 있다.
    - application-local.yml
        - 로컬 환경에서 사용하는 정보들은 application-local.yml 파일에 설정한다.
    - application-server.yml
        - 서버 환경에서 사용하는 정보들은 application-server.yml 파일에 설정한다.
        - 대표적인 서버 환경의 설정 정보 : DB 접속 정보
<br>

2. [Edit Run/Debug Configuration] 다이얼로그를 오픈하기 위해 애플리케이션 실행 파일이 위치한 셀렉트 박스를 클릭한 후, [Edit Configurations]를 클릭한다.
<br>

3. [Program arguments] 필드에 --spring.profiles.active=local을 입력해서 활성화할 프로파일을 ‘local’로 지정한다. 여기서 ‘local’은 application-local.yml 파일명에서의 ‘local’을 가리킨다.
    - Spring에서 프로파일을 지정하는 가장 쉬운 방법은 '-(대시)'를 기준으로 프로파일명을 yml 파일 이름 안에 포함하는 것이다.
<br>

4. 프로파일이 정상적으로 적용되었는지 확인하기 위해 애플리케이션 실행 후, 로그를 확인한다.
<br>

#### 빌드된 실행 파일에 프로파일 적용
IntelliJ IDE 같은 개발 환경에서 테스트를 위해 프로파일을 변경해야 할 경우도 있지만 프로파일 기능은 빌드된 실행 파일을 어느 환경에서 실행할지 여부를 결정할 때 주로 사용된다.
<br>

실행 파일을 실행시키는 `java -jar Jar_파일명.jar`에 IntelliJ IDE에서 Program arguments에 추가했던 것과 동일하게 설정을 추가하면 된다.

- 예시

```
`java -jar Jar_파일명.jar --Spring.profiles.active=local`
```

<br>

### 애플리케이션 배포
#### 전통적인 배포 방법
Spring Boot 기반의 Executable Jar 파일을 서버에 배포하는 가장 일반적인 방법은 scp나 sftp 같은 표준 유닉스 툴을 이용해서 서버로 전송하는 것이다.
서버로 전송된 Jar 파일은 JVM이 설치된 환경이라면 어디서든 쉽게 실행할 수 있다.
<br>

#### 클라우드 서비스를 위한 배포 방법
Executable Jar 파일은 특히 클라우드 환경에 손쉽게 배포할 수 있다.

- Paas(Platform as a Service)
    - Cloud Foundry, Heroku
    - 대표적인 Paas 제공 기업인 Cloud Foundry에서 제공하는 cf command line 툴을 사용하면 Executable Jar 파일을 손쉽게 배포할 수 있다.
    - 예시
    
```
$ cf push acloudyspringtime -p target/app-0.0.1-SNAPSHOT.jar
```

- IaaS(Infrastructure as a Service)
    - Executable Jar는 AWS Elastic Beanstalk, AWS Container Registry, AWS Code Deploy 같은 서비스를 이용해서 손쉽게 배포가 가능하다.
    - Microsoft의 클라우드 서비스인 Azure는 Azure Spring Cloud, Azure App Service에서 Spring Boot 기반의 Executable Jar 파일 배포 기능을 제공한다.
    - Google Cloud는 Executable Jar 파일 배포를 위한 여러 가지 옵션을 제공하고 있다.
    
- CI/CD 플랫폼을 사용한 배포
    - Executable Jar 파일에 대한 배포 자동화를 이루고 싶다면 Github Actions나 Circle CI 같은 CI / CD 플랫폼을 이용해 AWS나 Azure 같은 클라우드 서비스에 Executable Jar 파일을 자동 배포하도록 구성할 수 있다.