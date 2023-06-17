태그: #Cloud 
연결문서: [Cloud - 운영 환경 구성 4](Cloud%20-%20운영%20환경%20구성%204.md)

# EC2 인스턴스 상에서 서버 실행

## 1. 인스턴스에 개발 환경 구축하기
- 컴퓨터 운영체제를 처음 구입하면 필요한 프로그램을 설치해야 하듯이, EC2 인스턴스에 처음 접속하면 서버를 구동하는 데 필요한 개발 환경을 구축하는 것부터 시작해야 함
- EC2 인스턴스와 연결한 터미널에서 아래 명령어를 입력하며, 패키지 매니저가 관리하는 패키지의 정보를 최신 상태로 업데이트하기 위해서 아래 명령어를 사용함
<br>

```bash
sudo apt update
```

<br>

- 업데이트 과정이 끝나면 java를 설치해야 함
<br>

```bash
sudo apt install openjdk-11-jre-headless
```

<br>

- 아래와 같은 확인창이 나올 경우 "Y"를 입력함
- 설치 과정이 마무리되면, java -version 명령어를 입력하여 java 라이브러리가 설치가 완료되었는지 확인함
<br>

```bash
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libasound2 libasound2-data libgraphite2-3 libharfbuzz0b
Suggested packages:
  libasound2-plugins alsa-utils libnss-mdns fonts-dejavu-extra fonts-ipafont-gothic fonts-ipafont-mincho fonts-wqy-microhei
  | fonts-wqy-zenhei fonts-indic
The following NEW packages will be installed:
  libasound2 libasound2-data libgraphite2-3 libharfbuzz0b openjdk-11-jre-headless
0 upgraded, 5 newly installed, 0 to remove and 70 not upgraded.
Need to get 37.9 MB of archives.
After this operation, 173 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```

<br>

---

<br>

## 2. git을 통해 코드 클론 받기
- 코드가 저장된 깃헙 레포지토리 주소를 복사하고, git clone 명령어를 통해 EC2 인스턴스에 실습 코드를 클론 받음
- clone을 위해서는 SSH 등록이 필요함
- 관리해야 할 EC2의 수가 늘어나게 경우
    - GitHub에 등록된 SSH의 수가 EC2의 개수만큼 늘어나게 될 것이므로, EC2의 SSH 정보를 삭제하여 관리해야 함
<br>

- SSH 등록을 마쳤다면 인스턴스 시작 시 기본 경로가 아닌 홈 경로(~)로 이동하여 clone함
<br>

```bash
mason@ip-address:/var/snap/amazon-ssm-agent/1234 $ cd ~
mason@ip-address:~$ git clone [깃허브 클론 주소]
Cloning into '깃허브 레포지토리 명'...
Username for 'https://github.com': mason
Password for 'https://mason@github.com:
...
```

<br>

- 클론 진행 여부에 대한 질문이 나올 경우 yes를 입력하여 코드를 클론함
- 정상적으로 클론 했는지 확인하기 위해 터미널에 ls 명령어를 입력함
- 코드 폴더명이 보이면 정상적으로 다운로드가 완료된 것으로, 터미널을 통해 코드 안의 서버 코드 디렉토리로 이동함
<br>

```bash
cd [깃허브 레포지토리명]/[서버 코드 디렉토리]
```

<br>

- 빌드작업을 진행함
- 정상적으로 빌드가 완료되었으면 터미널에 ls명령어를 입력하면, build폴더를 확인할 수 있음
<br>

```bash
./gradlew build
```

<br>

---

<br>

## 3. EC2 인스턴스에서 서버 실행하기

- 아래 명령어를 이용하여 빌드된 파일을 실행함
<br>

```bash
java -jar build/libs/[서버 코드 디렉터리명]-0.0.1-SNAPSHOT.jar
```

<br>

- 정상적으로 서버가 실행된 것이 확인되면, EC2 인스턴스의 IP 주소로 접근해서 테스트를 진행함
- IP 주소는 EC2 대시보드에서 생성한 EC2 인스턴스를 클릭하면 확인할 수 있음
<br>

![](image_62.png)

<br>

- 아래에서 보라색으로 강조된 부분을 보시면, 두 가지 형태의 주소가 존재하는 것을 확인할 수 있음
    - 퍼블릭 IPv4 주소와 퍼블릭 IPv4 DNS는 형태만 다를 뿐 같은 주소
    - 둘 중 어떤 주소를 사용하셔도 문제가 없음
- 주소를 복사하여 주소창에 넣어 확인하는 게 아니라 개방 주소법 링크를 이용해 확인하는 경우 EC2 인스턴스 주소가 https로 연결될 수 있음
- 애플리케이션이 실행 중인데 실행 화면이 보이지 않는 경우 http로 변경하여 진행해야 함
<br>

![](image_63.png)

<br>

- EC2 인스턴스의 IP 주소로 접속하면 아래와 같은 화면을 볼 수 있음
    - EC2 인스턴스의 보안 그룹이 설정되어 있어야 에어 없이 바로 접근이 가능함
    - 프로젝트가 실행 중일 때 터미널에 Ctrl + C 단축키를 통해 실행 중인 프로젝트를 강제종료 할 수 있음
<br>

![](image_64.png)