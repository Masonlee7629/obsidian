태그: #Cloud 
연결문서: [Cloud - 운영 전략 1](Cloud%20-%20운영%20전략%201.md)

# Github Actions를 통한 배포 자동화

## 빌드파일 배포 및 실행

### CodeDeploy 설정
- S3에 저장된 빌드 파일을 EC2 인스턴스로 전달하기 위해 CodeDeploy 설정을 진행함

<br>

#### CodeDeploy 애플리케이션 생성하기
- 배포 > 애플리케이션을 클릭함
- [애플리케이션 생성] 버튼을 클릭함
- 애플리케이션 이름을 작성하고, 컴퓨팅 플랫폼은 EC2/온프레미스를 선택함
- 애플리케이션 이름의 경우 Github Actions 워크플로에 작성이 필요함
- 애플리케이션이 생성되면 해당 애플리케이션 내에 배포 그룹을 생성함

<br>

#### 배포 그룹 생성하기
- 배포 그룹은 방금 전 생성한 애플리케이션에 생성되어야 함
    - 다른 애플리케이션에 생성하고 있거나 애플리케이션 정보가 잘못된 경우 이전 진행 내역을 다시 진행해야 함
    - 배포 그룹 이름 역시 Github Actions 워크플로에 작성이 필요함
    - 애플리케이션과 구분할 수 있도록 규칙(e.g. be-0-GithubUsername-group)에 맞게 네이밍해야 함
- CodeDeploy 의 권한 부여를 위한 IAM 역할은 IAM Role을 연결함
- EC2 인스턴스의 태그를 이용해 배포 그룹 환경을 구성함
    - 최하단의 로드밸런서 설정에서 [로드 밸런싱 활성화] 체크를 해제함
    - 다른 설정은 기본값을 유지하고 배포 그룹을 생성함

<br>

#### .yml 파일 설정하기
- Github Actions 워크플로를 수정하기 전에 Code Deploy의 작동을 모아놓은 appspec.yml 파일을 설정해야 함
<br>

- 최상위 디렉토리 구조에 appspec.yml 파일을 생성함
<br>

```java
version: 0.0
os: linux
files:
  - source:  /
    destination: /home/ubuntu/action
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ubuntu
    group: ubuntu

hooks:
  ApplicationStart:
    - location: scripts/deploy.sh
      timeout: 60
      runas: ubuntu
```

<br>

#### shell script 생성하기
- scripts 폴더를 생성한 후 디렉토리 내부에 deploy.sh 파일을 생성함
- 해당 쉘 스크립트는 EC2 배포 진행 상황 별 로그를 기록하고 새로 배포된 빌드 파일을 실행함
<br>

```bash
#!/bin/bash
# 빌드 파일의 이름이 콘텐츠와 다르다면 다음 줄의 .jar 파일 이름을 수정해야 함
BUILD_JAR=$(ls /home/ubuntu/action/build/libs/githubAction-deploy-0.0.1-SNAPSHOT.jar)
JAR_NAME=$(basename $BUILD_JAR)

echo "> 현재 시간: $(date)" >> /home/ubuntu/action/deploy.log

echo "> build 파일명: $JAR_NAME" >> /home/ubuntu/action/deploy.log

echo "> build 파일 복사" >> /home/ubuntu/action/deploy.log
DEPLOY_PATH=/home/ubuntu/action/
cp $BUILD_JAR $DEPLOY_PATH

echo "> 현재 실행중인 애플리케이션 pid 확인" >> /home/ubuntu/action/deploy.log
CURRENT_PID=$(pgrep -f $JAR_NAME)

if [ -z $CURRENT_PID ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않음" >> /home/ubuntu/action/deploy.log
else
  echo "> kill -9 $CURRENT_PID" >> /home/ubuntu/action/deploy.log
  sudo kill -9 $CURRENT_PID
  sleep 5
fi


DEPLOY_JAR=$DEPLOY_PATH$JAR_NAME
echo "> DEPLOY_JAR 배포"    >> /home/ubuntu/action/deploy.log
sudo nohup java -jar $DEPLOY_JAR >> /home/ubuntu/action/deploy.log 2>/home/ubuntu/action/deploy_err.log &
```

<br>

### CodeDeploy 연결하기
- gradle.yml 파일을 수정하여 워크플로의 하단에 Code Deploy 배포 명령을 추가함
- CodeDeploy를 설정하며 작성한 애플리케이션의 이름, 배포 그룹의 이름을 표시된 곳에 각각 작성함
- 수정을 마치고 레포지토리에 push 하면 Github Actions가 실행됨
<br>

```java
name: Java CI with Gradle

on:
  push:
    branches: [ "main" ]

permissions:
  contents: read
  
env:
  S3_BUCKET_NAME: be-mason-project

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
      with:
        arguments: build
    
    # build한 후 프로젝트를 압축합니다.
    - name: Make zip file
      run: zip -r ./practice-deploy.zip .
      shell: bash
    
    # Access Key와 Secret Access Key를 통해 권한을 확인합니다.
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-northeast-2
    
    # 압축한 프로젝트를 S3로 전송합니다.
    - name: Upload to S3
      run: aws s3 cp --region ap-northeast-2 ./practice-deploy.zip s3://$S3_BUCKET_NAME/practice-deploy.zip
    
    # CodeDeploy에게 배포 명령을 내립니다.
    - name: Code Deploy
      run: >
        aws deploy create-deployment --application-name be-mason-project
        --deployment-config-name CodeDeployDefault.AllAtOnce
        --deployment-group-name be-mason-project-group
        --s3-location bucket=$S3_BUCKET_NAME,bundleType=zip,key=practice-deploy.zip
```

<br>

---

<br>

## 배포 결과 및 로그 확인

### 배포 결과 확인
- Github Actions가 실행되고 워크플로, CodeDeploy배포가 모두 성공했다면 배포 결과를 확인할 수 있음
- EC2 인스턴스를 클릭하고 퍼블릭 DNS (혹은 퍼블릭 IP)로 접속하여 접근이 가능한지 확인함
- Whitelabel Error Page가 보인다면 배포에 성공했음을 알 수 있음
    - 스프링 프로젝트 생성 후 포트 번호를 변경하지 않았다면 8080번 포트까지 함께 작성해야 함
- EC2 인스턴스에서도 배포 및 프로젝트 실행 여부를 확인할 수 있음

<br>

### 배포 로그 확인
- 생성한 deploy.sh에는 배포가 진행될 때마다 상황을 기록하도록 작성되어있음
<br>

#### EC2 인스턴스에서 배포 로그 확인하기
- 배포를 마치면 action 디렉토리 내에 두 개의 로그 파일이 생성됨
- EC2 인스턴스에서의 빌드 과정은 deploy.log 파일에, 빌드 파일을 정상적으로 실행하지 못한다면 deploy_err.log 파일에 기록됨
- 에디터 혹은 cat 명령어로 로그를 확인할 수 있음

#### 파일 수정 후 배포 자동화 테스트하기
- 간단하게 스프링 프로젝트를 수정 후 push 까지 진행하게 되면 지금까지 설정한 배포 자동화 과정이 실행됨
- 파일 수정 후 진행된 배포 과정에서 새로운 로그가 작성되었음을 확인할 수 있음
- 처음 배포할 때와 다른점은, 이전에 배포된 빌드 파일이 실행중인 경우 이를 종료하고 업데이트된 빌드파일을 실행한다는 것
- 다시 EC2의 주소로 접근하면 배포 내용이 반영된, 새로 작성한 응답 페이지를 확인할 수 있음

<br>

### 애플리케이션이 실행되지 않을 때
- 각 단계의 로그를 이용해 어디서 문제가 발생했는지 유추할 수 있음
<br>

- 이전 작업 내용이 보이는 경우
    - 프로세스를 종료(kill)하고 Github Actions를 다시 시도함
- EC2 인스턴스에 빌드 결과물이 없는 경우
    - S3 버킷에 저장 내용 확인하고 CodeDeploy의 배포 내역을 점검함
- EC2 인스턴스에 빌드 결과물이 있는 경우
    - EC2 인스턴스에 기록된 로그를 확인함

<br>

---

<br>

## 자주 하는 질문(FAQ)

### 파이프라인을 삭제하고 다시 만드려고 하는데 서비스 역할 이름이 이미 존재하는 경우
- 파이프라인을 처음 생성했을 때 파이프라인 이름에 따라 서비스 역할을 이미 생성해서 발생하는 에러
- 파이프라인을 삭제한 후 다시 생성할 때는 새 서비스 역할이 아닌 기존 서비스 역할로 진행해야 함

<br>

### EC2 인스턴스 연결 버튼이 로딩만 되고 연결이 되지 않는 경우
- EC2 인스턴스 역할 부여에서 IAM Role의 권한과 신뢰 관계를 수정했으며, 이때 신뢰 관계를 편집할 때 배열로 기존의 EC2에 Code Deploy까지 추가해야 함
- 추가가 아닌 변경을 하지 않았는지 다시 확인해야 함

<br>

### 파이프라인 / 배포 그룹 / 파라미터 스토어 변수 등 리소스 생성이 안되는 경우
1. 서울 외 다른 리전에서 진행하는 경우
    - AWS 콘솔의 에러 문구를 보면 Code Deploy의 새 Application을 생성하는 과정에서 발생한 에러임을 나타냄
    - 이때 ARN(Amazon Resource Name)에 생성하려는 리소스의 리전이 us-east-1이라고 적혀있음
    - 리전을 변경하여 다시 진행해야 함
2. 네이밍 규칙을 위반한 경우
    - 서울 리전에서 진행하지만, 리소스 네이밍 규칙에 위반한 경우
    - 리소스를 생성할 때 Code Deploy의 Application과 Application에서 생성한 배포 그룹(Deployment Group)의 이름 규칙이 잘못된 경우

<br>

### 파라미터 스토어 이용 시 Rate exceeded 에러가 발생하는 경우
- 파라미터 스토어에서 변수를 등록하거나 값을 수정할 때 발생하는 에러
- 파라미터 스토어 서비스에 일시적으로 요청이 급증하며 AWS에서 앞선 요청을 처리하는 도중 보여주는 것으로, 새로고침 후 변수 등록이 확인되었다면 별다른 과정 없이 다음 단계로 넘어가면 됨

<br>

### 파이프라인 배포 자동화 이후 로그인이 되지 않는 경우
- 배포 자동화를 진행한 경우 먼저 서버가 제대로 실행 중인지 확인이 필요함
    - 환경변수 설정 파일을 위주로 확인함
<br>

1. Client 코드의 .env에 서버 주소(EC2 인스턴스의 주소)가 포트를 포함하여 잘 적혀있는지 확인함
    - EC2 인스턴스를 재부팅한 적이 있다면 Public IP 주소가 변경되었으므로 .env 파일의 서버 주소값을 변경하고 다시 빌드하여 S3에 재 업로드 해야 함
2. 기존에 application.properties 파일에 저장했던 환경 변수들이 파라미터 스토어에 모두 제대로 생성되어 있는지 확인함
3. 파라미터 스토어에 작성된 정보에 기반하여 bootstrap.yml 파일이 제대로 작성되어 있고, Github Repository가 최신 상태인지 확인함
    - 파라미터 스토어에서 값을 읽어오는 시점에 일시적으로 순간 요청이 너무 많이 몰린 상태로 값을 제대로 읽어오지 못할 수 있음
    - 파이프라인을 다시 작동시켜야 함

<br>

### 파이프라인이나 Github Actions를 다시 실행하고 싶은 경우
- 현재 Source Stage에서 Github(Repository)를 사용하고 있으며, 배포 자동화를 위한 준비가 모두 끝났다면, 연결되어 있는 Repository의 코드가 변화가 감지되면(e.g. push) 파이프라인이나 Github Actions가 자동으로 실행됨
- AWS 콘솔에서 리소스가 변화된 경우에는(e.g. 파라미터 스토어 변수값 변경, Code Deploy 설정 변경 등) 코드의 변화가 아니기 때문에 파이프라인이나 Github Actions가 자동으로 실행되지 않음
    - 코드의 변화는 없지만 파이프라인을 다시 실행하고 싶을 때는 다음 단계를 통해 각 자동화 프로세스를 진행할 수 있음
    1. 파이프라인 다시 실행하기
        - 파이프라인 우측 상단의 변경 사항 릴리스 버튼 클릭함
    2. Gitbhu Actions 다시 실행하기
        - Github Actions의 가장 최근에 실행된 workflow 선택함
        - Re-run all jobs 버튼을 클릭하여 workflow 재실행함