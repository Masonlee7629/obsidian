태그: #Cloud 
연결문서: [Cloud - 배포 자동화 4](Cloud%20-%20배포%20자동화%204.md)

# Github Actions를 통한 배포 자동화

## 개요 및 사전 준비 사항

### 주의 사항
- EC2 인스턴스에 파이프라인 실습으로 인해 실행되고 있는 애플리케이션이 있다면 프로세스를 종료함
- 이전에 진행한 실습으로 생긴 리소스를 리소스 이름을 확인한 후 모두 삭제하고 진행함
    - CodeBuild 빌드 프로젝트
    - CodeDeploy 애플리케이션과 배포그룹
    - CodePipeline 파이프라인
    - EC2 인스턴스에 있는 이전 빌드 결과물
- 안내되어 있는 이름 규칙을 반드시 지켜야 함
    - 이름 규칙, 태그 등 안내에 따르지 않으면 리소스가 삭제될 수 있음
- 실습에서 안내하는 기능이나 리소스 외에 다른 기능을 사용하는 경우, 개인 Root 계정을 사용해야 함

### 프로젝트 사전 준비
- https://start.spring.io/ 를 통해 간단한 프로젝트를 생성하며, 배포 과정에 집중하기 위해 Spring Web 의존 모듈만 선택하여 프로젝트를 구성하여 진행함
- 버전은 SNAPSHOT이 표기되어있지 않은 2.X.X를 선택함
- 프로젝트의 이름으로 빌드 파일이 생성되며, 이후 배포 단계에서 빌드 파일을 실행해야 하기 때문에 설정 파일 구성에 신경 써서 진행해야 함

<br>

### S3 버킷 비우기
- S3를 저장소로서 사용함
<br>

1. 개인 버킷의 내용물을 삭제함
    - 삭제 버튼을 눌러 저장된 모든 객체를 영구 삭제함
2. 정적 웹 사이트 호스팅 비활성화

<br>

---

<br>

## Github Actions를 통한 배포 Flow

### Github Actions이란?
- GitHub Actions는 Github가 공식적으로 제공하는 빌드, 테스트 및 배포 파이프라인을 자동화할 수 있는 CI/CD 플랫폼
- 레포지토리에서 Pull Request 나 push 같은 이벤트를 트리거로 GitHub 작업 워크플로우(Workflow)를 구성할 수 있음
- 워크플로우는 하나 이상의 작업이 실행되는 자동화 프로세스로, 각 작업은 자체 가상 머신 또는 컨테이너 내부에서 실행됨
- 워크플로우는 .yml (혹은 .yaml) 파일에 의해 구성되며, 테스트, 배포 등 기능에 따라 여러 개의 워크플로우도 만들 수 있으며, 생성된 워크플로우는 .github/workflows 디렉토리 이하에 위치함
- 비공개 레포지토리의 경우 Github Actions가 작동할 때의 용량과 시간이 제한되어 있으며 공개 레포지토리는 무료로 사용 가능함

<br>

### Github Actions를 통한 배포 Flow

![](image_65.png)

<br>

#### Github Actions
- Github Actions는 설정 파일(.yml)에 따라 Github Repository에 특정 변동사항을 트리거로 작동됨
- main 브랜치에 push 하는 경우 작동하도록 설정한 후 진행
- Github Actions는 main 브랜치에 적용된 변동 사항을 기준으로 프로젝트를 빌드하며, 빌드를 마친 프로젝트를 AWS의 S3 버킷에 저장하고, Code Deploy에 S3에서 EC2로 배포 명령을 내림

<br>

#### S3
- Github Actions를 통한 배포 자동화에서는 저장소로써 사용함
- Github Actions에서 빌드한 결과물이 압축되어 S3으로 전송되고, 버킷에 저장됨

<br>

#### Code Deploy
- Github Actions에서 배포 명령을 받은 Code Deploy는 S3에 저장되어 있는 빌드 결과물을 EC2 인스턴스로 이동함
- 프로젝트 최상단에 위치한 appepec.yml 설정 파일에 의해 쉘 스크립트 등 단계에 따라 특정 동작을 함
- Code Deploy가 S3 버킷에서 EC2 인스턴스로 프로젝트를 이동할 수 있도록 EC2 인스턴스에 Code Deploy Agent의 설치가 필요함
- OS별, 버전별 설치 방법이 상이하기 때문에 반드시 공식문서를 참고하며 진행해야 함

<br>

#### EC2
- Code Deploy에 의해 빌드 과정을 거친 프로젝트가 EC2 인스턴스로 전달되고, .yml (설정 파일)과 .sh (쉘 스크립트)에 의해 각 배포 결과를 로그로 저장하며 빌드 파일(.jar)을 실행함
- 해당 과정이 원활히 진행되기 위해, EC2 인스턴스에 접속하여 알맞은 Code Deploy Agent의 설치와 JDK 11 버전 설치가 필요함

<br>

---

<br>

## 리소스 설정하기

### Github Actions 생성

#### Public Repository 생성하기
- 레포지토리가 비공개(Private)인 경우 Github Actions 사용에 제한이 있을 수 있으므로, 반드시 공개 레포지토리로 생성해야 함
- 생성한 레포지토리의 Owner는 개인 계정이어야 하며, 여러 Organixation에 소속되어 있는 경우, 레포지토리의 Owner를 확인해야 함
- 레포지토리를 생성한 후, 미리 만들어놓은 프로젝트를 업로드 함
<br>

- 레포지토리의 Actions 탭을 클릭함
- Github Actions탭에선 워크플로를 어떻게 생성할지 선택할 수 있음
- 상단의 [set up a workflow yourself] 를 클릭하여 빈 yml 설정파일로 워크플로를 설정할 수 있고, 추천 워크플로 구성을 선택하여 진행할 수 있음
    - Gradle 빌드가 기본으로 설정되어있는 추천 구성을 사용하여 시작할 것
    - 만약 화면의 추천 구성이 보이지 않는 경우 Search workflows 에 'Java with Gradle'을 검색하여 선택하거나, 빈 파일에서 구성할 수 있음
<br>

- 다른 부분은 수정하지 않고 오른쪽 상단의 [Commit changes...] 버튼을 눌러 업로드 함
    - 브라우저에서 수정사항을 Commit 하면 Push까지 진행되어 Github Actions가 바로 실행됨
    - 웹 브라우저에서 생성한 gradle.yml 파일을 로컬 환경에서도 변경하기 위해선 pull 명령어를 이용해 로컬 코드에 반영해야 함

<br>

- gradle.yml 파일이 생성됨과 동시에 워크플로에 작성되어있는 트리거(main 브랜치에 push)로 인해 Github Actions가 실행됨
- Actions 탭에서 생성된 워크플로의 각 단계별 진행상황을 확인할 수 있음
- 실행한 워크플로가 성공하면 초록색 체크가, 실패하면 빨간색 X 모양을 확인할 수 있음
- 진행 / 성공 / 실패 여부는 레포지토리에서 최근 커밋 내역과 함께 표시됨

<br>

### Github Action 수정
- Github Actions를 AWS 리소스와 연결함
- IAM User를 생성해야 하며, 로그인할 때의 계정이 아닌, 서비스와의 연결을 위한 액세스 키 방식으로 발급된 계정
<br>

#### Github Secret 등록하기
- Github Actions에서 워크플로를 실행하는 과정에서 액세스 키가 필요하지만, 공개되면 보안 이슈가 발생할 수 있음
    - Github Secret을 이용해 액세스 키 값을 저장한 후 사용할 예정
<br>

- 레포지토리의 Settings > Secrets > Actions 탭으로 이동한 후 [New repository secret] 버튼을 클릭
- Name에는 변수 이름을, Value엔 변수의 값을 저장함
- 저장해야하는 변수는 두가지로, IAM User를 생성할 때 볼 수 있는 액세스 키 ID 값과, 비밀 액세스 키 값을 각각 저장함
    - 한 번 저장하면 값의 일부만은 수정할 수 없으니 주의해야 함
    - 덮어쓰기는 가능하지만, 저장된 값을 확인하는 것은 불가능

<br>

#### Github Action 설정파일 수정하기
- gradle.yml 파일을 수정함
    - 저장된 secret의 이름이 같도록 수정해야 함
    - ecrets.AWS_ACCESS_KEY와 secrets.AWS_SECRET_ACCESS_KEY는 이전에 등록한 Github Secret에서 값을 불러오므로 키 값을 yml파일에 바로 적지 않음
    - ap-northeast-2 는 서울 리전을 의미함
    - IDE를 이용해 파일을 수정하는 경우 (pull 진행 이후) 반드시 push 까지 진행해야 Github Actions가 동작함
    - 워크플로를 처음 생성했을 때처럼 브라우저에서 바로 수정할 수도 있음
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
    
    # Access Key와 Secret Access Key를 통해 권한을 확인함
    # 아래 코드에 Access Key와 Secret Key를 직접 작성하지 않음
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }} # 등록한 Github Secret이 자동으로 불려옴
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }} # 등록한 Github Secret이 자동으로 불려옴
        aws-region: ap-northeast-2
    
    # 압축한 프로젝트를 S3로 전송함
    - name: Upload to S3
      run: aws s3 cp --region ap-northeast-2 ./practice-deploy.zip s3://$S3_BUCKET_NAME/practice-deploy.zip
```

<br>

- 워크플로가 성공적으로 완료되면 S3 버킷에 압축파일이 전송되었음을 확인할 수 있음
    - 만약 실패했다면 어느 단계에서 실패했는지 확인한 후 실패한 이유를 파악한 후 다시 진행해야 함

<br>

#### 주의 사항
- 운영 체제 문제로 '~/gradlew' is not executable. 에러가 발생할 수 있음
- 에러가 발생하는 경우 Build with Gradle 이전에 ./gradlew에 권한을 부여하는 단계를 추가함
- 줄 바꿈이 있을 땐 각 단계가 같은 깊이를 유지하도록 간격에 유의해야 함
<br>

```java
// 예시 코드
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Add permission // 추가
      run: chmod +x gradlew // 추가
    - name: Build with Gradle
```