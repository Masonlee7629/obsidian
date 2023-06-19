태그: #Cloud 
연결문서: [Cloud - 배포 자동화 3](Cloud%20-%20배포%20자동화%203.md)

# 배포 자동화

## 환경 변수 설정

### 환경 변수 생성 후 EC2에 전달
- 파이프라인에 올린 EC2 instance에 환경 변수를 전달하는 방법
- 비밀번호와 같은 환경 변수는 외부에 노출되면 안 되기 때문에, 소스코드에 포함할 수 없음
- AWS Parameter Store 서비스를 이용하면 EC2 instance에 환경 변수를 전달할 수 있음

<br>

1. Parameter Store 대시보드로 이동 후, [ 파라미터 생성 ] 버튼을 클릭함
2. 환경변수명과 값 입력 후, 우측 하단의 [ 파라미터 생성 ] 버튼을 클릭함
    - 운영 환경 구성 시 총 4개(spring.datasource.url, spring.datasource.username, spring.datasource.password, config.domain)의 환경 변수를 이용해 DB와의 연결을 세팅두었으며, 해당 환변 변수를 파라미터 스토어에 등록함
    - 이름에는 환경 변수명을 적어주고, 환경 변수에 할당되어야 할 값을 입력함
    - 이름은 꼭 /prefix/name/key 의 순서로 네이밍 규칙에 맞게 작성되어야 함
        - Ex) /spring-boot-aws/be-0-NAME/key
    - 이름의 key에 해당하는 이름이 application.properties에 작성한 spring.datasource.url, spring.datasource.username, spring.datasource.password, config.domain와 같은 변수명에 해당함
    - 원하는 유형으로 값을 저장할 수 있으며, 보안 문자열을 선택하면 브라우저에서 값을 확인할 수 없어 더 안전하게 관리할 수 있음
    - 값에는 각 변수(spring.datasource.url, spring.datasource.username, spring.datasource.password, config.domain)에 할당할 값을 입력함
    - 운영 환경 구성과 마찬가지로 중괄호 { } 안에는 적절한 값을 입력해야 함
        - spring.datasource.url의 경우 중괄호 안에 포트를 포함해 RDS 인스턴스 엔드포인트가 작성되어야 함
    - 입력이 완료되면, 우측 하단으로 내려가 [ 파라미터 생성 ] 버튼을 눌러 파라미터를 생성함
    - 해당 과정을 반복해 총 4개의 파라미터를 모두 생성해야 함
<br>

3. 로컬 환경에서 소스 코드의 build.gradle 파일을 수정함
<br>

```java
plugins {
	id 'org.springframework.boot' version '2.5.4'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.mason.project'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web',
			'io.jsonwebtoken:jjwt-api:0.11.2'
	implementation 'org.springframework.cloud:spring-cloud-starter-aws-parameter-store-config' // 추가
	implementation 'junit:junit:4.13.1'
	runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.2',
			'io.jsonwebtoken:jjwt-jackson:0.11.2'
	compileOnly "com.fasterxml.jackson.core:jackson-databind:2.9.4"
	compileOnly 'org.projectlombok:lombok'
	compileOnly "org.springframework.boot:spring-boot-starter-security"
	runtimeOnly 'mysql:mysql-connector-java'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement { // 블록 추가
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-starter-parent:Hoxton.SR12"
	}
}

test {
	useJUnitPlatform()
}
```

<br>

4. src/main/resources/bootstrap.yml 파일을 생성함
<br>

```java
aws:
  paramstore:
    enabled: true
    prefix: /spring-boot-aws
    name: 리소스_이름
    profileSeparator: _
```

<br>

5. src/main/resources/application.properties 파일을 수정함
    - 변수의 값을 여기에 작성하는 게 아니라 AWS Parameter Store에서 가져오게 됨
    - 변수의 이름이 잘 작성되었는지, 파라미터 스토어에 올바른 값이 작성되었는지 확인해야 함
    - 변경 사항이 있다면 수정 후 저장하고 commit한 뒤, main으로 push하여 파이프라인을 통해 변경 사항을 전달함
    - 배포 자동화에서의 환경변수는 깃허브 레포지토리에 올리는게 아니라 별도의 방식으로 관리가 필요함
<br>

```java
spring.jpa.database=mysql
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
#spring.datasource.url=jdbc:mysql://{AWS RDS Endpoint}/test?useSSL=false&characterEncoding=UTF-8&serverTimezone=UTC
#spring.datasource.username={RDS Mysql Admin id}
#spring.datasource.password={RDS Mysql Admin password}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#config.domain={AWS S3 Endpoint}
```