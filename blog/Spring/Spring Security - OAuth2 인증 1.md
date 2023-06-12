태그: #Spring 
연결문서: [Spring Security - OAuth2 인증 2](Spring%20Security%20-%20OAuth2%20인증%202.md)

# OAuth2(Authentication)

## OAuth2란?
- 특정 애플리케이션(Client)에서 사용자의 인증을 직접 처리하는 것이 아니라 사용자 정보를 보유하고 있는 신뢰할 만한 써드 파티 애플리케이션(GitHub, Google, Facebook 등)에서 사용자의 인증을 대신 처리해 주고 Resource에 대한 자격 증명용 토큰을 발급한 후, Client가 해당 토큰을 이용해 써드 파티 애플리케이션의 서비스를 사용하게 해주는 방식

<br>

### OAuth 2를 사용하는 애플리케이션 유형
1. 써드 파티 애플리케이션에서 제공하는 API의 직접적인 사용
    - Google, Github, Facebook 같은 신뢰할만한 써드 파티 애플리케이션에서 제공하는 API를 직접적으로 사용하는 애플리케이션을 구현하는데 OAuth 2를 사용할 수 있음
    - 사용자가 OAuth 2 인증 프로토콜을 이용해 써드 파티 애플리케이션에 대한 인증에 성공하면 써드 파티 애플리케이션에서 제공하는 API를 활용한 커스텀 서비스를 제공하는 것
2. 추가적인 인증 서비스 제공 용도
    - 써드 파티 애플리케이션의 서비스를 이용하는 것뿐만 아니라 추가적인 인증 서비스를 제공하기 위한 용도로 사용하기 위함
    - 일반적으로 제공하는 아이디/패스워드 로그인 인증 이외에 OAuth 2를 이용한 로그인 인증 방법을 추가적으로 제공하는 것
    - 만약 특정 서비스를 제공하는 애플리케이션에서 사용자의 크리덴셜(Credential)을 남기고 싶지 않을 경우 OAuth 2 로그인 인증 방법으로 로그인하면 됨

<br>

---

<br>

## OAuth2의 동작 방식

### OAuth2 인증 컴포넌트(Component, 구성요소)들의 역할
- Resource Owner
    - Resource Owner는 사용하고자 하는 Resource의 소유자를 말함
    - Resource Owner는 한마디로 서비스를 이용하는 사용자
- Client
    - Resource Owner를 대신해 보호된 Resource에 액세스하는 애플리케이션
    - Client는 서버, 데스크톱, 모바일 또는 기타 장치에서 실행되는 애플리케이션이 될 수 있음
    - Client와 Server에 대한 기준을 잡기 위해서는 서비스를 제공하는 쪽을 기준으로 생각해야 함
- Resource Server
    - OAuth 2 인증 처리 흐름 상에서의 Resource Server는 Client의 요청을 수락하고 Resource Owner에게 해당하는 Resource를 제공하는 서버
- Authorization Server
    - OAuth 2 인증 처리 흐름에서 Authorization Server는 Client가 Resource Server에 접근할 수 있는 권한을 부여하는 서버
    - Resource Owner가 인증에 성공하면 Authorization Server는 Client에게 Access Token 형태로 Resource Owner의 Resource에 접근할 수 있는 권한을 부여함

<br>

### OAuth 2 컴포넌트 간의 상호 작용을 통한 인증 처리 흐름

![](image_20.png)

<br>

1. Resource Owner는 Client 역할을 하는 웹 애플리케이션에게 OAuth2 인증을 요청
    - 인증 요청은 Resource Owner가 인증을 할 수 있도록 Client인 웹 애플리케이션이 자체적으로 로그인 화면을 제공하는 것이 아님
    - Resource Owner는 자신의 계정 정보를 관리하고 있는 써드 파티 애플리케이션에 로그인하길 원하는 것이며, 이 요청을 Client인 웹 애플리케이션에 전송하는 것
2. Client는 Resource Owner가 Resource Owner의 계정 정보를 관리하고 있는 써드 파티 애플리케이션에 로그인할 수 있도록 써드 파티 애플리케이션의 로그인 페이지로 리다이렉트(Redirect) 함
3. Resource Owner는 로그인 인증을 진행하고 로그인 인증에 성공
4. Authorization Server가 Resource Owner의 로그인 인증이 성공적으로 수행되었음을 증명하는 Access Token을 Client에게 전송함
    - Resource Owner에게 Access Token을 전송하는 것이 아니라 Client 애플리케이션에게 전송하는 것
    - Client는 Resource Owner를 대신해서 Resource Owner의 Resource를 사용하는 대리인 역할을 하기 때문
5. Access Token을 전달받은 Client는 이제 Resource Owner의 대리인 역할을 수행할 수 있게 되었으므로, Resource Server에게 Resource Owner 소유의 Resource를 요청함
6. Resource Server는 Client가 전송한 Access Token을 검증해서 Client가 Resource Owner의 대리인으로서의 자격이 증명되면 Resource Owner의 Resource를 Client에게 전송함

<br>

### OAuth 2 인증 프로토콜에서 사용되는 용어
- Authorization Grant
    - Client 애플리케이션이 Access Token을 얻기 위한 Resource Owner의 권한을 표현하는 크리덴셜(Credential)을 의미함
    - Client가 Access Token을 얻기 위한 수단이라고 생각하면 되며 네 가지 Authorization Grant 타입이 있음
        - Authorization Code
        - Implicit Grant Type
        - Client Credentials
        - Resource Owner Password Credentials
- Access Token
    - Client가 Resource Server에 있는 보호된 Resource에 액세스하기 위해 사용하는 자격 증명용 토큰
    - Authorization Code와 Client Secret을 이용해 Authorization Server로부터 전달받은 Access Token으로 자격을 증명하면 Resource Server에 접근할 수 있음
- Scope
    - 주어진 액세스 토큰을 사용하여 액세스할 수 있는 Resource의 범위를 의미함
    - 예를 들어 Scope 설정을 통해 해당 Resource Owner의 사진첩에는 접근할 수 있지만, 연락처 등 다른 Resource에는 접근할 수 없도록 접근 권한을 지정할 수도 있음

<br>

### Authorization Grant 유형

#### 1. Authorization Code Grant : 권한 부여 승인 코드 방식
- 권한 부여 승인을 위해 자체 생성한 Authorization Code를 전달하는 방식으로, 가장 많이 쓰이고 기본이 되는 방식
- Authorization Code Grant 방식에서는 Refresh Token을 사용할 수 있음
- 권한 부여 승인 요청 시 응답 타입(response_type)을 code로 지정하여 요청함
<br>

![](image_21.png)

<br>

1. Resource Owner는 소셜 로그인 버튼을 누르는 등의 서비스 요청을 Client(애플리케이션)에게 전송함
2. Client는 Authorization Server에 Authorization Code를 요청하며, 이때 미리 생성한 Client ID, Redirect URI, 응답 타입을 함께 전송함
3. Resource Owner는 로그인 페이지를 통해 로그인을 진행함
4. 로그인이 확인되면 Authorization Server는 Authorization Code를 Client에게 전달함
    - 이 전에 요청과 함께 전달한 Redirect URI로 Code를 전달함
5. Client는 전달받은 Authorization Code를 이용해 Access Token 발급을 요청함
    - AccessToken을 요청할 때 미리 생성한 Client Secret, Redirect URI, 권한 부여 방식, Authorization Code를 함께 전송함
6. 요청 정보를 확인한 후 Redirect URI로 Access Token을 발급함
7. Client는 발급받은 Access Token을 이용해 Resource Server에 Resource를 요청함
8. Access Token을 확인한 후 요청받은 Resource를 Client에게 전달함

<br>

#### 2. Implicit Grant : 암묵적 승인 방식
- 별도의 Authorization Code 없이 바로 Access Token을 발급하는 방식
    - 자격증명을 안전하게 저장하기 힘든 Client(자바스크립트 등 스크립트 언어를 사용하는 브라우저)에게 최적화된 방식
    - Refresh Token 사용이 불가능하며, 이 방식에서 Authorization Server는 Client Secret을 통해 클라이언트 인증 과정을 생략함
- 권한 부여 승인 요청 시 응답 타입(response_type)을 token으로 지정하여 요청함
<br>

![](image_22.png)

<br>

1. Resource Owner는 소셜 로그인 버튼을 누르는 등의 서비스 요청을 Client(애플리케이션)에게 전송함
2. Client는 Authorization Server에게 접근 권한을 요청함
    - 요청과 함께 미리 생성한 Client ID, Redirect URI, 응답 타입을 전송함
3. Resource Owner는 로그인 페이지를 통해 로그인을 진행함
4. 로그인이 확인되면 Authorization Server는 Client에게 Access Token을 전달함
5. Client는 Access Token을 이용해 Resource Server에게 Resource를 요청함
6. Access Token을 확인한 후 요청받은 Resource를 전달함

<br>

#### 3. Resource Owner Password Credential Grant : 자원 소유자 자격 증명 승인 방식
- 간단하게 로그인 시 필요한 정보(username, password)로 Access Token을 발급받는 방식
    - 자신의 서비스에서 제공하는 애플리케이션의 경우에만 사용되는 인증 방식으로, Refresh Token의 사용도 가능함
    - Authorization Server, Resource Server, Client가 모두 같은 시스템에 속해 있을 때만 사용이 가능함
<br>

![](image_23.png)

<br>

1. Resource Owner는 로그인 버튼을 누르는 등의 서비스 요청을 Client(애플리케이션)에게 전송하며, 이때 로그인에 필요한 정보(Username, Password)를 이용해 요청함
2. Client에서는 Resource Owner에게서 전달받은 로그인 정보를 통해 Authorization Server에 Access Token을 요청하며, 미리 생성한 Client ID, 권한 부여 방식, 로그인 정보를 함께 전달함
3. 요청과 함께 온 정보들을 확인한 후 Client에게 Access Token을 전달함
4. Client는 Access Token을 이용하여 Resource Server에게 Resource를 요청함
5. Access Token을 확인한 후 요청받은 Resource를 전달함

<br>

#### 4. Client Credentials Grant : 클라이언트 자격 증명 승인 방식
- Client 자신이 관리하는 Resource 혹은 Authorization Server에 해당 Client를 위한 제한된 Resource 접근 권한이 설정되어 있는 경우 사용할 수 있는 방식
- 이 방식은 자격 증명을 안전하게 보관할 수 있는 Client에서만 사용되어야 하며, Refresh Token의 사용은 불가능함
<br>

![](image_24.png)