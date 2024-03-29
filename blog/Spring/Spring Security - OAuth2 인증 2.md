태그: #Spring 
연결문서: [Spring Security - OAuth2 인증 3](Spring%20Security%20-%20OAuth2%20인증%203.md)

# Spring Security에서의 OAuth2 인증

## OAuth2 인증을 위한 사전 작업

### 구글 API 콘솔에서의 OAuth2 설정
- 구글의 OAuth 2 시스템을 이용하기 위해서는 먼저 구글 API 및 서비스 콘솔에서 OAuth 2 시스템을 이용하기 위한 클라이언트 ID와 Secret을 생성해야 함

<br>

#### 1. 프로젝트 생성

![](image_25.png)

<br>

- 메인 화면의 프로젝트 만들기를 클릭함
    - 만약 링크가 보이지 않는다면 상단의 프로젝트 선택을 클릭한 후, 오픈된 팝업에서 새 프로젝트 링크를 클릭함

<br>

![](image_26.png)

<br>

- 새로운 프로젝트를 만듦
    - 프로젝트의 이름을 입력하고, 나머지 항목을 디폴트 항목으로 사용하면 됨
    - 생성한 프로젝트는 만들어지기까지 1분 내외의 시간을 기다려야 함

<br>

![](image_27.png)

<br>

- 사용 설정된 API 및 서비스 상단에 프로젝트 선택 메뉴를 클릭하고 생성한 프로젝트를 선택함

<br>

#### 2. OAuth 동의 화면 만들기

![](image_28.png)

<br>

- 생성된 프로젝트를 선택하면 API 및 서비스 대시보드를 확인할 수 있음
    - 왼쪽 메뉴에서 OAuth 동의 화면을 클릭함

<br>

![](image_29.png)

<br>

- OAuth 동의 화면에서 User Type을 외부로 선택 체크한 뒤, 만들기 버튼을 클릭함

<br>

![](image_30.png)

<br>

- 앱 등록 수정 화면에서 ‘앱 이름’과 ‘사용자 지원 이메일’, 개발자 연락처 정보에서 ‘이메일 주소’를 입력한 후 저장 후 계속 버튼을 클릭함

<br>

![](image_31.png)

<br>

- 테스트 사용자 화면에서 저장 후 계속 버튼을 클릭함

<br>

#### 3. 사용자 인증 정보 생성

![](image_32.png)

<br>

- 왼쪽 목록에서 사용자 인증 정보를 클릭함

<br>

![](image_33.png)

<br>

- 사용자 인증 정보 만들기를 클릭한 후, OAuth 클라이언트 ID를 클릭함

<br>

![](image_34.png)

<br>

- 애플리케이션 유형은 ‘웹 애플리케이션’을 선택함
- 애플리케이션의 이름을 입력함
- 승인된 리디렉션 URI에 http://localhost:8080/login/oauth2/code/google를 입력함
- 만들기 버튼을 클릭함

<br>

![](image_35.png)

<br>

- OAuth 클라이언트가 생성되었다는 화면이 보인다면 그림과 같이 클라이언트 ID와 클라이언트 보안 비밀번호(Secret)를 확인할 수 있음
- 클라이언트 ID와 클라이언트 보안 비밀번호(Secret)는 Spring Security 기반의 애플리케이션의 설정 정보로 사용되므로 안전하게 잘 보관해두어야 함