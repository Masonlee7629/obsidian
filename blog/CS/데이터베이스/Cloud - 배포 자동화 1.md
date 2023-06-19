태그: #Cloud 
연결문서: [Cloud - 배포 자동화 2](Cloud%20-%20배포%20자동화%202.md)

# 배포 자동화

## Automated Deployment

### 배포 자동화
- 배포 자동화란 한 번의 클릭 혹은 명령어 입력을 통해 전체 배포 과정을 자동으로 진행하는 것을 뜻함
- 배포 자동화가 필요한 이유
    - 수동적이고 반복적인 배포 과정을 자동화함으로써 시간이 절약됨
    - 휴먼 에러(Human Error)를 방지할 수 있음

<br>

### 배포 자동화 파이프라인
- 배포에서 파이프라인(Pipeline)이란 용어는 소스 코드의 관리부터 실제 서비스로의 배포 과정을 연결하는 구조를 뜻함
- 파이프라인은 전체 배포 과정을 여러 단계(Stages)로 분리하며, 각 단계는 파이프라인 안에서 순차적으로 실행되며, 각 단계마다 주어진 작업(Actions)들을 수행함
- 파이프라인을 여러 단계로 분리할 때, 대표적으로 쓰이는 세 가지 단계가 존재함
    1. Source 단계 : Source 단계에서는 원격 저장소에 관리되고 있는 소스 코드에 변경 사항이 일어날 경우, 이를 감지하고 다음 단계로 전달하는 작업을 수행함
    2. Build 단계 : Build 단계에서는 Source 단계에서 전달받은 코드를 컴파일, 빌드, 테스트하여 가공하며, Build 단계를 거쳐 생성된 결과물을 다음 단계로 전달하는 작업을 수행함
    3. Deploy 단계 : Deploy 단계에서는 Build 단계로부터 전달받은 결과물을 실제 서비스에 반영하는 작업을 수행함
- 파이프라인의 단계는 상황과 필요에 따라 더 세분화되거나 간소화될 수 있음

<br>

### AWS 개발자 도구
- AWS 에는 개발자 도구 섹션이 존재하며, 개발자 도구 섹션에서 제공하는 서비스를 활용하여 배포 자동화 파이프라인을 구축할 수 있음
<br>

#### CodeCommit
- Source 단계를 구성할 때 CodeCommit 서비스를 이용함
- GitHub과 유사한 서비스를 제공하는 버전 관리 도구
- GitHub과 비교했을 때 CodeCommit 서비스는 보안과 관련된 기능에 강점을 가지고 있지만, CodeCommit을 사용할 때는 과금 가능성을 고려해야 하며, 프리티어 한계 이상으로 사용할 시 사용 요금이 부과될 수도 있음

<br>

#### CodeBuild
- Build 단계에서는 CodeBuild 서비스를 이용함
- CodeBuild 서비스를 통해 유닛 테스트, 컴파일, 빌드와 같은 빌드 단계에서 필수적으로 실행되어야 할 작업들을 명령어를 통해 실행할 수 있음

<br>

#### CodeDeploy
- Deploy 단계를 구성할 때는 기본적으로 다양한 서비스를 이용할 수 있음
- CodeDeploy 서비스를 이용하면 실행되고 있는 서버 애플리케이션에 실시간으로 변경 사항을 전달할 수 있음
- S3 서비스를 통해 S3 버킷을 통해 업로드된 정적 웹 사이트에 변경 사항을 실시간으로 전달하고 반영할 수 있음

<br>

#### CodePipeline
- 각 단계를 연결하는 파이프라인을 구축할 때 CodePipeline 서비스를 이용함
- AWS 프리티어 계정 사용 시 한 계정에 두 개 이상의 파이프라인을 생성하면 추가 요금이 부여될 수 있음

<br>

---

<br>

## AWS Pipeline을 통한 배포 자동화

### EC2 인스턴스 역할 부여
1. EC2 대시보드로 이동 후, 인스턴스로 이동함
2. 배정받은 인스턴스를 검색하여 선택한 후 [ 태그 ] 를 확인함
    - 태그는 Key - Value 쌍의 값으로, 공용으로 할당된 cohort 태그와 개인 소유 리소스를 식별하기 위한 Name 태그가 있음
    - Name 태그의 경우 인스턴스 이름에 적용됨
    - 태그는 후에 있을 파이프라인 구축 단계에서 인스턴스를 잘 식별하기 위해 사용함
3. 배정받은 인스턴스를 선택한 후 [ 보안 ] -> [ IAM 역할 ] 을 확인함
    - 배정받은 리소스 이름과 비교하여 본인 소유의 역할이 맞는지 확인한 후 클릭함
    - 인스턴스에 연결되어있는 IAM 역할을 클릭하면, 해당 역할에 현재 연결되어있는 권한과 역할 등 다양한 정보를 확인할 수 있음
4. 오른쪽 [ 권한 추가 ] → [ 정책 연결 ] 을 클릭함
5. 검색창에 S3을 검색하여 AmazonS3FullAccess를 선택함
6. 동일한 방법으로 AmazonEC2RoleforAWSCodeDeploy 를 선택함
7. 동일한 방법으로 AWSCodeDeployRole 를 선택함
8. AmazonSSMFullAccess를 선택하고 우측 하단의 [ 정책 연결 ] 버튼을 클릭함
9. 정책이 연결되었다는 안내와 보이면 하단의 권한을 확인함
10. 신뢰 관계를 편집함
    - 신뢰 관계란, 해당 역할을 취할 수 있는 서비스나 사용자를 명시하는 부분
    - access 정책 부여를 통해 역할을 생성해주었지만, access 할 수 있는 서비스를 신뢰 관계에서 명시함으로써 역할이 확실히 완성됨
11. 처음 편집 화면으로 들어가면 "Service"의 값으로 "ec2.amazonaws.com"만 할당되어 있음
12. "Service" 값을 대괄호를 작성하고 내부에 "codedeploy.ap-northeast-2.amazonaws.com" 값을 추가한 후, 하단의 [ 정책 업데이트 ]를 클릭함

<br>

### EC2를 활용한 파이프라인 구축
- 로컬 환경의 서버코드 디렉토리 경로에 appspec.yml 파일을 추가함
    - appspec.yml은 배포 자동화를 도와주는 CodeDeploy-Agent가 인식하는 파일
    - appspec.yml 코드 스닙펫을 이용해 내용을 채움

<br>

#### appspec.yml
- CodeDeploy가 읽는 파일
- files의 destination을 보면 파이프라인의 결과물이 EC2의 어느 위치에 복사될지 알 수 있음
- hooks는 CodeDeploy에서 지정한 각 단계에 맞춰 어떤 셸 스크립트를 실행하는지 지정함
<br>

```md
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ubuntu/build

hooks:
  BeforeInstall:
    - location: server_clear.sh
      timeout: 3000
      runas: root
  AfterInstall:
    - location: initialize.sh
      timeout: 3000
      runas: root
  ApplicationStart:
    - location: server_start.sh
      timeout: 3000
      runas: root
  ApplicationStop:
    - location: server_stop.sh
      timeout: 3000
      runas: root
```

<br>

#### buildspec.yml
- CodeBuild가 읽는 파일
- CodeBuild가 지정한 각 단계에 맞춰 동작을 특정하여 명령함
<br>

```md
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto11
  build:
    commands:
      - echo Build Starting on `date`
      - cd DeployServer
      - chmod +x ./gradlew
      - ./gradlew build
  post_build:
    commands:
      - echo $(basename ./DeployServer/build/libs/*.jar)
artifacts:
  files:
    - DeployServer/build/libs/*.jar
    - DeployServer/scripts/**
    - DeployServer/appspec.yml
  discard-paths: yes
```

<br>

#### scripts/ 하위 셸 스크립트
- scripts 디렉토리를 생성한 후 네 가지 셸 스크립트를 생성함
- 디렉토리 이름과 파일 이름에 주의하여 생성해야 함
<br>

- scripts/initialize.sh
    - 빌드 결과물을 실행할 수 있도록 실행 권한을 추가함
<br>

```md
#!/usr/bin/env bash
chmod +x /home/ubuntu/build/**
```

<br>

- scripts/server_clear.sh
    - 빌드 결과물이 저장되어 있는 build 디렉토리를 제거함
<br>

```md
#!/usr/bin/env bash
rm -rf /home/ubuntu/build
```

<br>

- scripts/server_start.sh
    - [서버 코드 디렉토리]-0.0.1-SNAPSHOT.jar라는 빌드 결과물을 실행
<br>

```md
#!/usr/bin/env bash
cd /home/ubuntu/build
sudo nohup java -jar [서버 코드 디렉토리]-0.0.1-SNAPSHOT.jar > /dev/null 2> /dev/null < /dev/null &
```

<br>

- scripts/server_stop.sh
    - 실행 중인 Spring Boot 프로젝트를 종료함
<br>

```md
#!/usr/bin/env bash
sudo pkill -f 'java -jar'
```

<br>

---

<br>

## 서버 배포 자동화 결과 확인
- Postman 혹은 브라우저를 이용해 테스트를 진행함
- 생성한 EC2 인스턴스의 IP 주소를 이용하여 테스트를 진행함
- EC2 인스턴스를 재시작한 경우 IP 주소가 변경되므로 [클라이언트 코드 디렉토리] > index.js 파일의 url 변수 값의 수정 후 객체 업로드 과정이 필요함