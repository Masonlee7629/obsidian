태그: #Cloud 
연결문서: [Cloud - 배포 자동화 1](Cloud%20-%20배포%20자동화%201.md)

# Docker

## Docker CLI
- Docker Image 및 Container를 다루기 위한 Docker CLI(Command Line) 명령어
- Ubuntu 운영체제로 진행하는 경우, 관리자 권한(sudo)으로 Docker 명령어를 실행해야 함

<br>

### Docker CLI(Command Line Interface)
- 도커를 이용하는 데 있어서 명령어, 옵션 등 사용법은 Docker docs에서 확인할 수 있음
- Docker CLI 관련 정보뿐만 아니라 Docker의 전반적인 사용법과 환경을 구성하는 방법에 대해서도 확인할 수 있
    - 사용법 : Docker CLI, Docker-Compose CLI, API Reference
    - 환경 및 빌드 파일 구성 : DockerFile, Docker-Compose File

<br>

---

<br>

## 도커 이용하기

### docker/whalesay
- docker/whalesay라는 이미지로 예제를 실습함
- 먼저, 제공된 이미지를 읽을 수 있어야 하며, docker/whalesay는 레지스트리 계정, 레포지토리 이름, 태그 세 가지 정보로 구성되어 있음
    - 레지스트리(Registry)
        - Docker Hub : https://hub.docker.com/
        - 도커 이미지를 관리하는 공간
        - 특별하 다른 것을 지정하지 않는다면, 도커 허브(Docker Hub)를 기본 레지스트리로 설정함
        - 레지스트리는 Docker Hub, Private Docker Hub, 회사 내부용 레지스트리 등으로 나뉠 수 있음
    - 레포지토리(Repository)
        - 레지스트리 내에 도커 이미지가 저장되는 공간
        - 이미지 이름이 사용되기도 함
        - Github의 레포지토리와 유사함
    - 태그(Tag)
        - 같은 이미지라고 할지라도 버전 별로 안의 내용이 조금 다를 수 있음
        - 해당 이미지를 설명하는 버전 정보를 주로 입력함
        - 특별히 다른 것을 지정하지 않는다면 latest 태그를 붙인 이미지를 가져옴
- docker/whalesay:latest라는 문장은 다음과 같이 읽을 수 있음
    - Docker Hub라는 레지스트리에서
    - docker라는 유저가 등록한 whalesay 이미지 혹은 레포지토리에서
    - latest 태그를 가진 이미지

<br>

#### Docker Example 수행하기 : docker/whalesay
- docker/whalesay의 최신 이미지를 받아옴
- image pull : 레지스트리에서 이미지 혹은 레포지토리를 가져옴
<br>

```bash
docker image pull docker/whalesay:latest
```

<br>

- 이미지 리스트를 출력함
<br>

```bash
docker image ls
```

<br>

- 받아온 이미지를 실행함(이미지 -> 컨테이너)
- 컨테이너 실행 시 작성한 명령어의 구성
    - {container} run
        - 컨테이너를 실행함
    - [OPTIONS]
        - -name : 컨테이너의 이름을 할당함
    - [COMMAND]
        - command는 초기 컨테이너 실행 시 수행되는 명령어
        - cowsay : 컨테이너 실행 시 cowsay 명령어를 호출하며, node를 호출하듯 이용함
    - [ARG..]
        - boo : COMMAND인 cowsay에 넘겨질 파라미터
<br>

```bash
docker container run --name 컨테이너_이름 docker/whalesay:latest cowsay boo
```

<br>

- 모든 컨테이너의 리스트를 출력함
    - {container} ps : 컨테이너의 리스트를 출력함
    - a : Default로는 실행되는 컨테이너지만 종료된 컨테이너를 포함하여 모든 컨테이너를 출력함
<br>

```bash
docker container ps -a
```

<br>

- 컨테이너_이름이라는 이름을 가진 컨테이너를 삭제함
    - {container} rm : 컨테이너를 지칭해서 삭제함
        - 컨테이너를 명시할 때는 ps 명령을 통해 확인할 수 있는 NAMES 혹은 CONTAINER ID를 사용함
<br>

```bash
docker container rm 컨테이너_이름
```

<br>

- docker image의 용량 확인과 docker/whalesay 이미지를 삭제
<br>

```bash
# docker image 의 용량 확인
docker image ls

# docker/whalesay 이미지 지우기
docker image rm docker/whalesay
```

<br>

- 세 가지 작업을 한 번에 실행할 경우
    - {container} run : 컨테이너를 실행하며, 이미지가 없다면 이미지를 받아온 뒤(pull) 실행함
    - -rm : 컨테이너를 일회성으로 실행하며, 컨테이너가 중지되거나 종료될 때, 컨테이너와 관련된 리소스를 모두 제거함
<br>

```bash
docker container run --name 컨테이너_이름 --rm docker/whalesay cowsay boo
```

<br>

---

<br>

## Docker CLI - Copy, Dockerfile
- 다른 사람이 제공한 도커 이미지를 받아 사용하는 경우, 원하는 모든 기능이 구성되어 있지 않을 수 있음
- 도커 이미지에 파일을 추가하고, 도커 이미지를 만드는 방법이 존재함
- 로컬에 저장된 파일과 함께 도커 이미지를 이용하는 방식이며, 로컬 파일과 도커 이미지를 연결하는 방법 중 CP를 이용해 로컬 파일을 이미지에 복사하고, 팩맨 게임 서버를 실행하는 작업을 수행함

<br>

### Docker 컨테이너에 파일을 복사하기
- 사용할 모든 파일이 하나의 이미지에 구성되어 있고, 그 이미지를 사용하는 방법과 달리, 게임 서버, 웹 서버와 같이 사용할 도구가 도커 이미지에 모두 구성되어 있지 않은 경우도 있음
    1. 웹 서버는 도커 컨테이너로 실행
    2. 웹 서버를 구성하는 파일은 직접 만들거나 가져온 파일 구성
<br>

- 장점
    - 서버에 문제가 생기는 것을 호스트와 별개로 파악할 수 있음
    - 문제가 생긴 서버를 끄고, 마치 공장 초기화를 하듯 도커 이미지로 서버를 재구동할 수 있음
<br>

- 로컬에 있는 파일과 도커 이미지를 연결하는 방법은 크게 CP(Copy)를 이용하는 방법과 Docker Volume 기능을 이용하는 방법으로 나뉨
    - CP(Copy) : 호스트와 컨테이너 사이에 파일을 복사(Copy)
    - Volume : 호스트와 컨테이너 사이에 공간을 마운트(Mount)
        - 마운트는 저장 공간을 다른 장치에서 접근할 수 있도록 경로를 허용해서, 마치 하나의 저장 공간을 이용하는 것처럼 보이게 하는 작업을 말함

<br>

### httpd 웹 서버 컨테이너 실행하기
- 동작, 그림, 사운드 파일 모두를 웹 서버에 업로드해서 사용함

<br>

#### httpd 웹 서버
- 사용할 도커 이미지는 httpd(http daemon)
- httpd(http daemon)는 Apache HTTP Server를 실행할 수 있는 오픈소스 웹 서버 소프트웨어
    - httpd는 /usr/local/apache2/htdocs/ 경로에 웹 서버와 관련된 파일들이 저장되어 있다면, 해당 파일을 기반으로 웹 서버가 실행되도록 함
- WSL 터미널 혹은 Ubuntu 운영체제를 통해 진행한다면, 모든 명령어 앞에 sudo 명령어를 붙여야 함

<br>

1. 동작, 그림, 사운드 파일이 있는 레포지토리를 클론함
<br>

```bash
git clone git@github.com:[레포지토리 주소]
혹은
git clone https://github.com/[레포지토리 주소]
```

<br>

2. docker container run 명령어로 httpd를 실행함
    - run의 옵션을 [docker container run](https://docs.docker.com/engine/reference/commandline/container_run/)에서 다시 확인할 수 있음
    - -p 옵션은 로컬호스트의 포트와 컨테이너의 포트를 연결하며, 2의 명령어에서 818 포트가 로컬호스트의 포트이고, 80번은 컨테이너의 포트
    - httpd는 일정 시간 연결 기록이 없으면, 서버 가동이 중지되며, 실행 중이던 도커 컨테이너가 중지되었다면, 다시 실행함
    - 터미널에서 1, 2의 명령어를 입력했을 때, 터미널의 작동이 중단된 것처럼 보여도 정상적인 상태이므로 터미널을 종료하지 말고, 다른 터미널 창을 열어 계속해서 진행함
    - -d 옵션은 컨테이너를 백그라운드에서 실행하게 해주는 옵션
<br>

```bash
docker container run --name 컨테이너_이름 -p 818:80 httpd
```

<br>

3. 127.0.0.1:818 혹은 localhost:818을 통해 웹 서버가 작동하고 있는지 확인함
    - 127.0.0.1과 localhost를 이용하면 로컬 컴퓨터의 IP 주소로 redirecting할 수 있음
    - DNS 설정으로 인해 localhost로 접속이 안 되고 127.0.0.1만 접속이 될 수 있음

<br>

4. 서버가 정상적으로 열린 것을 확인한 후, 새로운 터미널을 열어 docker container cp 명령어를 입력해 로컬호스트에 있는 파일을 컨테이너에 전달함
    - 경로를 입력할 때, 상대 경로와 절대 경로를 주의해서 작업해야 함
    - docker container cp 명령은 앞 경로의 파일을 뒤 경로에 복사함
    - 명령어를 입력할 땐 명령어와 경로 간 띄어쓰기에 주의해서 입력해야 함
<br>

```bash
docker container cp ./ 컨테이너_이름:[로컬호스트 파일 경로]
```

<br>

5. 127.0.0.1:818 혹은 localhost:818에 접속해서 서버가 구동되는지 확인함

<br>

### Docker 이미지 만들기
- Docker Container를 이미지 파일로 변환하는 것으로, 이미지로 만들어 놓을 때의 장점은 다음과 같음
    - 이전에 작업했던 내용을 다시 한번 수행하지 않아도 됨
    - 배포 및 관리가 유용함

<br>

#### 1. 구동한 Docker Container를 이미지로 만드는 방법
- docker container commit 명령을 이용함
<br>

```bash
docker container commit 컨테이너_이름 이미지_이름
```

<br>

- 생성된 이미지를 900 포트에서 웹 서버로 구동함
<br>

```bash
docker run --name my_web2 -p 900:80 이미지_이름
```

<br>

- 127.0.0.1:900 혹은 localhost:900을 통해 웹 서버가 작동하고 있는지 확인함

<br>

#### 2. Docker Image 빌드를 위한 파일인 Dockerfile로 만드는 방법
- Dockerfile을 만들고, Dockerfile 대로 이미지를 build하는 방법
- Dockerfile은 이미지 파일의 설명서라고 할 수 있음
<br>

```bash
FROM httpd:2.4 # 베이스 이미지를 httpd:2.4로 사용함
WORKDIR [컨테이너 내의 작업 디렉토리] # (Optional) 컨테이너 내의 작업 디렉토리를 설정함
COPY ./ [복사할 위치의 디렉터리] # 호스트의 현재 경로(./)에 있는 파일을 생성할 이미지를 복사할 위치의 디렉터리에 복사함
```

<br>

- Dockerfile을 수정하고 알맞은 경로를 작성하여 저장함
<br>

```bash
FROM httpd:2.4
COPY {웹 서버 실행에 필요한 파일의 경로 1} 복사할_위치의_디렉터리_경로 # 띄어쓰기에 유의하여 작성함
COPY {웹 서버 실행에 필요한 파일의 경로 2} 복사할_위치의_디렉터리_경로
```

<br>

- docker build 명령은, Dockerfile로 도커 이미지 파일을 생성함
<br>

```bash
# --tag 는 name:tag 형식으로 이미지를 생성할 수 있음
# 지정한 경로에 있는 Dockerfile을 찾아서 빌드함
docker build --tag 이미지_이름 . # "."을 명령어에 꼭 포함해야 함 
```

<br>

- 생성된 이미지를 이용해 901 포트에 웹 서버 구동함
<br>

```bash
docker run --name my_web3 -p 901:80 이미지_이름
```

<br>

- 127.0.0.1:901 혹은 localhost:901을 통해 게임 서버가 작동하고 있는지 확인함