태그: #Spring 
연결문서: [Spring MVC - Spring Rest Docs 2](Spring%20MVC%20-%20Spring%20Rest%20Docs%202.md)

# Spring Rest Docs

## Spring Rest Docs란?

Spring Rest Docs는 REST API 문서를 자동으로 생성해 주는 Spring 하위 프로젝트이다.  
  

Spring Rest Docs의 가장 큰 특징은 Controller의 슬라이스 테스트를 통해 테스트가 통과 되어야지만 API 문서가 정상적으로 만들어 진다는 것이다.

### Spring Rest Docs의 API 문서 생성 흐름

[##_Image|kage@m4yuu/btsaR0PA52v/mykfHQzijtMN8e79raerA0/img.png|CDM|1.3|{"originWidth":500,"originHeight":1080,"style":"alignCenter","width":null}_##]

1.  **테스트 코드 작성**
    1.  슬라이스 테스트 코드 작성
        -   Spring Rest Docs는 Controller에 대한 슬라이스 테스트 코드를 먼저 작성한다.
    2.  API 스펙 정보 코드 작성
        -   슬라이스 테스트 코드 다음으로 Controller에 정의 되어 있는 API 스펙 정보(Request Body, Response Body, Query Parameter 등)를 코드로 작성한다.
2.  **test 태스크(task) 실행**
    1.  작성된 슬라이스 태스크 코드를 실행한다.
        -   하나의 테스트 클래스를 실행시켜도 되지만 일반적으로 Gradle의 빌드 태스트(task)중 하나인 test task를 실행 시켜서 API 문서 스니핏(snippet)을 일괄 생성한다.
    2.  테스트 실행 결과가 "`passed`"이면 다음 작업을 진행하고, "`failed`"이면 문제를 해결하기 위해 테스트 케이스를 수정한 후 다시 테스트를 진행해야 한다.
3.  **API 문서 스니핏(.adoc 파일) 생성**
    -   테스트 실행 결과가 "`passed`"이면 테스트 코드에 포함된 API 스펙 정보 코드를 기반으로 API 문서 스니핏이 `.adoc` 확장자를 가진 파일로 생성된다.
        -   스니핏(Snippet) : 일반적으로 코드의 일부 조각을 의미하는 경우가 많은데, 여기서는 **문서의 일부 조각**을 의미한다. **테스트 케이스 하나 당 하나의 스피닛이 생성**되며, 여러 개의 스니핏을 모아서 하나의 API 문서를 생성할 수 있다.

### Spring Rest Docs 설정

Spring Rest Docs가 API 문서 생성 작업을 정상적으로 수행할 수 있도록 기본적인 설정 작업을 먼저 해 주어야 한다.

1.  build.gradle 설정
2.  API 문서 스니핏을 사용하기 위한 템플릿 API 문서 생성

#### build.gradle 설정

```
plugins {
    id 'org.springframework.boot' version '2.7.1'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id "org.asciidoctor.jvm.convert" version "3.3.2"    // (1)
    id 'java'
}

...
...

repositories {
    mavenCentral()
}

// (2)
ext {
    set('snippetsDir', file("build/generated-snippets"))
}

// (3)
configurations {
    asciidoctorExtensions
}

dependencies {
        // (4)
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'

        // (5) 
    asciidoctorExtensions 'org.springframework.restdocs:spring-restdocs-asciidoctor'

    ...
    ...
}

// (6)
tasks.named('test') {
    outputs.dir snippetsDir
    useJUnitPlatform()
}

// (7)
tasks.named('asciidoctor') {
    configurations "asciidoctorExtensions"
    inputs.dir snippetsDir
    dependsOn test
}

// (8)
task copyDocument(type: Copy) {
    dependsOn asciidoctor            // (8-1)
    from file("${asciidoctor.outputDir}")   // (8-2)
    into file("src/main/resources/static/docs")   // (8-3)
}

build {
    dependsOn copyDocument  // (9)
}

// (10)
bootJar {
    dependsOn copyDocument    // (10-1)
    from ("${asciidoctor.outputDir}") {  // (10-2)
        into 'static/docs'     // (10-3)
    }
}
```

1.  (1)에서는 `.adoc` 파일 확장자를 가지는 AsciiDoc 문서를 생성해주는 Asciidoctor를 사용하기 위한 플러그인을 추가한다.
2.  (2)에서는 `ext` 변수의 `set()` 메서드를 이용해서 API 문서 스니핏이 생성될 경로를 지정한다.
3.  (3)에서는 Asciidoctor에서 사용되는 의존 그룹을 지정한다. asciidoctor task가 실행되면 내부적으로 (3)에서 지정한 `asciidoctorExtensions`라는 그룹을 지정한다.
4.  (4)에서 'org.springframework.restdocs:spring-restdocs-mockmvc'를 추가함으로써 spring-restdocs-core와 spring-restdocs-mockmvc 의존 라이브러리가 추가된다.
5.  (5)에서 spring-restdocs-asciidoctor 의존 라이브러리를 추가한다. (3)에서 지정한 `asciidoctorExtensions` 그룹에 의존 라이브러리가 포함된다.
6.  (6)에서는 test task 실행 시, API 문서 생성 스니핏 디렉토리 경로를 설정한다.
7.  (7)에서는 asciidoctor task 실행 시, Asciidoctor 기능을 사용하기 위해 asciidoctor task에 `asciidoctorExtensions`을 설정한다.
8.  (8)은 build task 실행 전에 실행되는 task로, copyDocument task가 수행되면 index.html 파일이 `src/main/resources/static/docs`에 copy되며, copy된 index.html 파일은 API 문서를 파일 형태로 외부에 제공하기 위한 용도로 사용할 수 있다.
    -   (8-1)에서는 asciidoctor task가 실행된 후에 task가 실행되도록 의존성을 설정한다.
    -   (8-2)에서는 "`build/docs/asciidoc/`" 경로에 생성되는 index.html을 copy한다.
    -   (8-3)의 "`src/main/resources/static/docs`" 경로로 copy된 index.html을 추가해준다.
9.  (9)에서는 build task가 실행되지 전에 copyDocument task가 먼저 수행되도록 한다.
10.  (10)에서는 애플리케이션 실행 파일이 생성하는 bootJar task 설정이다.
    -   (10-1)에서는 bootJar task 실행 전에 copyDocument task가 실행되도록 의존성을 설정한다.
    -   (10-2)와 (10-2)에서는 Asciidoctor 실행으로 생성되는 index.html 파일을 jar 파일 안에 추가한다. jar 파일에 index.html을 추가함으로써 웹 브라우저에서 접속(`http://localhost:8080/docs/index.html`) 후, API 문서를 확인할 수 있다.

#### API 문서 스니핏을 사용하기 위한 템플릿(또는 source 파일) 생성

build.gradle에 API 문서 생성을 위한 설정이 완료되었다면, 마지막으로 API 문서 스니핏이 생성 되었을 때, 이 스니핏을 사용해서 최종 API 문서로 만들어주는 템플릿 문서(`index.adoc`)를 생성해야 한다.

-   Gradle 기반 프로젝트에서는 아래 경로에 해당하는 디렉토리를 생성해주어야 한다.
    -   `src/docs/asciidoc/`
-   다음으로 `src/docs/asciidoc/` 디렉토리 내에 비어있는 템플릿 문서(`index.adoc`)를 생성해주어야 한다.

해당 사전 준비가 완료되면, 테스트 케이스에 API 스펙 정보를 추가해주면 API 문서 스니핏을 생성할 수 있다.