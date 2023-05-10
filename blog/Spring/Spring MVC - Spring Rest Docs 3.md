태그: #Spring 
연결문서: [Spring MVC - 애플리케이션 빌드, 실행, 배포](Spring%20MVC%20-%20애플리케이션%20빌드,%20실행,%20배포.md)

# Spring Rest Docs

## 스니핏을 이용한 API 문서화

### API 문서 템플릿 생성을 위한 디렉토리 및 템플릿 문서 생성

API 문서 스니핏은 문서 일부에 포함되는 조각 모음이다.  
  

이 조각 모음을 제대로 된 문서로 만들기 위해서는 스니핏을 포함하는 템플릿 문서가 필요하다.  
  

"`index.adoc`"파일을 아직 생성하지 않았다면 “`src/docs/asciidoc`” 디렉토리를 생성하고 비어 있는 “index.adoc” 파일을 생성한다.

### 템플릿 문서 내용 추가

```
= xxxx 애플리케이션    // (1)
:sectnums:
:toc: left                
:toclevels: 4
:toc-title: Table of Contents
:source-highlighter: prettify

Lee Dae Gyeom <mason@mason.com>   // (2)

v1.0.0, 2023.04.16    // (3)

// (4)
***
== Controller
=== 회원 등록
.curl-request
include::{snippets}/post-member/curl-request.adoc[]     

.http-request
include::{snippets}/post-member/http-request.adoc[]

.request-fields
include::{snippets}/post-member/request-fields.adoc[]

.http-response
include::{snippets}/post-member/http-response.adoc[]

.response-headers
include::{snippets}/post-member/response-headers.adoc[]

=== 회원 정보 수정
.curl-request
include::{snippets}/patch-member/curl-request.adoc[]

.http-request
include::{snippets}/patch-member/http-request.adoc[]

.path-parameters
include::{snippets}/patch-member/path-parameters.adoc[]

.request-fields
include::{snippets}/patch-member/request-fields.adoc[]

.http-response
include::{snippets}/patch-member/http-response.adoc[]

.response-fields
include::{snippets}/patch-member/response-fields.adoc[]
```

위 코드는 Asciidoc 문법으로 작성된 템플릿 문서이다.

1.  (1)은 API 문서의 제목이다.
2.  (2)는 API 문서를 생성한 생성자의 정보이다.
3.  (1)과 (2) 사이에 있는 항목은 API 문서의 목차와 관련되 내용이다.
4.  (3)은 API 문서의 생성 날짜이다.
5.  (4)부터는 테스트 케이스 실행을 통해 생성한 API 문서 스니핏을 사용하는 부분이다.

템플릿 문서에서 스니핏을 사용하는 방법은 정해져 있다.  
`include::{snippets}/스니핏 문서가 위치한 디렉토리/스니핏 문서파일명.adoc[]`

6.  문서 작성이 완료되면 Gradle의 `:build` 또는 `:bootJar` task 명령을 실행해서 `index.adoc` 파일을 `index.html` 파일로 변환하면 된다.  
    변환된 파일은 `src/main/resources/static/doc` 디렉토리에 생성된다.

---

## Spring Rest Docs에서의 Asciidoc

### Asciidoc

Asciidoc은 Spring Rest Docs를 통해 생성되는 텍스트 기반 문서 포맷이다.  
  

Asciidoc 포맷을 사용해서 메모, 문서, 기사, 서적, E-Book, 웹 페이지, 매뉴얼 페이지, 블로그 게시물 등을 작성할 수 있으며 Asciidoc 포맷으로 작성된 문서는 HTML, PDF, EPUB, 매뉴얼 페이지를 포함한 다양한 형식으로 변환될 수 있다.  
  

Asciidoc은 주로 **기술 문서 작성을 위해 설계된 가벼운 마크업 언어**이기도 하다.

#### 목차 구성

```
= xxxx 애플리케이션    // (1)
:sectnums:             // (2)
:toc: left           // (3)     
:toclevels: 4         // (4)
:toc-title: Table of Contents  // (5)
:source-highlighter: prettify  // (6)

Lee Dae Gyeom <mason@mason.com>

v1.0.0, 2023.04.16
```

Asciidoc 문법으로 목차를 구성하는 방법은 다음과 같다.

1.  (1)에 문서의 제목을 작성하기 위해 `=`을 추가한다.  
    `====`과 같이 `=`의 개수가 늘어날수록 글자 크기는 작아진다.
2.  (2)는 목차에서 각 섹션에 넘버링을 해주기 위한 부분으로 `:sectnums:`를 추가한다.
3.  (3)의 `:toc:`는 목차를 문서의 어느 위치에 구성할 것인지를 설정하는 부분으로, 여기서는 문서의 왼쪽에 목차가 표시되도록 `left`를 지정했다.
4.  (4)의 `:toclevels:`은 목차에 표시할 제목의 level을 지정한다. 여기서는 4로 지정했기 때문에 `====`까지의 제목만 목차에 표시된다.
5.  (5)의 `:toc-title:`은 목차의 제목을 지정할 수 있다.
6.  (6)의 `:source-highlighter:`은 문서에 표시되는 소스 코드 하이라이터를 지정한다. 여기서는 `prettify`를 지정했다.

#### 박스 문단 사용하기

```
***     // (1)
API 문서 개요
 // (2)
 이 문서는 Spring MVC 기반의 REST API 기반 애플리케이션에 대해 학습하는 블로그에 관한 내용이다.

***
```

위와 같이 API 문서에 박스 문단을 구성해서 API 문서에 대한 설명을 추가할 수 있다.  
  

(1)의 `***`는 단락을 구분 지을 수 있는 수평선을 추가한다.  
(2)와 같이 문단의 제목 다음에 한 라인을 띄우고 한 칸 들여쓰기의 문단을 작성하면 박스 문단을 사용할 수 있다.

#### 경고 문구 추가

```
***
API 문서 개요

 이 문서는 Spring MVC 기반의 REST API 기반 애플리케이션에 대해 학습하는 블로그에 관한 내용이다.

 // (1)
CAUTION: 이 문서는 학습용으로 일부 기능에 제한이 있습니다. 기능 제한 사항에 대해 알고 싶다면 담당자에게 문의 하세요
***
```

(1)과 같이 `CAUTION:` 을 사용해서 경고 문구를 추가할 수 있다. 이 외에 `NOTE:`, `TIP:`, `IMPORTANT:`, `WARNING:` 등을 사용할 수 있다.

#### URL Scheme 자동 인식

다음과 같은 URL Scheme는 Asciidoc에서 자동으로 인식하여 링크가 설정된다.

-   http
-   https
-   ftp
-   irc
-   mailto
-   [mason@gmail.com](mailto:mason@gmail.com)

#### 이미지 추가

```
image::https://spring.io/images/spring-logo-9146a4d3298760c2e7e49595184e1975.svg[spring]
```

API 문서에 이미지를 추가하고 싶다면 `image::` 를 사용해서 추가할 수 있다.

### Asciidoctor

Asciidoctor는 AsciiDoc 포맷의 문서를 파싱해서 HTML 5, 매뉴얼 페이지, PDF 및 EPUB 3 등의 문서를 생성하는 툴이다.  
  

Spring Rest Docs에서는 Asciidoc 포맷의 문서를 HTML 파일로 변환하기 위해 내부적으로 Asciidoctor를 사용하고 있다.

#### 문서 스니핏을 템플릿 문서에 포함하는 방법

이는 Asciidoctor가 템플릿 문서 변환 작업을 수행하기 이 전 단계이다.

```
***
== Controller
=== 회원 등록
.curl-request       // (1)
include::{snippets}/post-member/http-request.adoc[]    // (2)

.request-fields
include::{snippets}/post-member/request-fields.adoc[]

.http-response
include::{snippets}/post-member/http-response.adoc[]

.response-fields
include::{snippets}/post-member/response-fields.adoc[]

...
...
```

1.  (1)의 `.curl-request` 에서 `.`은 하나의 스니핏 섹션 제목을 표현하기 위해 사용한다. `curl-request` 은 섹션의 제목이며, 원하는 대로 수정하면 된다.
2.  (2)에서 `include`는 Asciidoctor에서 사용하는 매크로(macro) 중 하나이며, 스니핏을 템플릿 문서에 포함할 때 사용한다.  
    `::` 은 매크로를 사용하기 위한 표기법이며, `{snippets}`는 해당 스니핏이 생성되는 디폴트 경로를 의미하며, 아래의 `build.gradle` 파일에 설정한 snippetsDir 변수를 참조하는데 사용할 수 있다.

-   Asciidoctor에서는 어떤 작업을 처리하기 위한 용어로 매크로(macro)라는 용어를 사용한다.

```
// build.gradle 파일
ext {
    set('snippetsDir', file("build/generated-snippets"))
}
```
