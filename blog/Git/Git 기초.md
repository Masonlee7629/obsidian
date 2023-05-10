태그: #Git

# Git 이란?

Git 이란 쉽게 말해 파일을 관리해주는 프로그램

-   파일의 변경 사항을 추적하며, 사용자가 각 파일의 버전을 관리할 수 있도록 한다
-   파일의 백업을 지원한다
-   협업자들과 파일을 공유하고, 각자의 작업물을 취합할 수 있도록 한다

## Git 과 GitHub 란?

일반적으로 Git 자체는 로컬에서 버전을 관리해주는 프로그램이지만, 백업, 협업을 위한 기능을 활용하려면 온라인 원격 저장소가 필요. Github 는 이런 원격 저장소 기능을 제공해주는 서비스 중 하나

-   Git : 로컬에서 버전을 관리해주는 프로그램
-   GitHub : Git이 설치되어 있는 클라우드 저장소

## Git 을 이용한 기본 Workflow

Git 을 이용한 버전 관리 기능의 전체적인 흐름

1.  Remote 의 다른 Repository 에서 Fork 하여 Remote 의 내 Repository 에 가져온다
2.  내 PC 로 clone 하여 해당 코드를 가져와 수정한다
3.  내 PC 의 작업공간(Work Space) 에서 작업한 파일을 Add 한다
4.  Add 한 파일에 대한 내역을 Commit 한다
5.  Remote 의 내 Repository 에 Push 해서 수정된 파일과 Commit 을 기록한다
6.  원래의 Repository 에 Pull Request 를 보내 내가 수정한 코드를 업로드한다

## Git 의 기본 명령어

-   git init  
    : 로컬 Git 저장소를 설정(초기화)  
    : 해당 명령어를 입력 한 후, 추가적인 git 명령어를 사용 가능
-   git clone < 원격 저장소 URL >  
    : 원격 저장소로부터 프로젝트를 복제  
    : 저장소를 clone 하면 'origin'이란 리모트 저장소가 자동으로 등록
-   git remote -v  
    : 현재 프로젝트에 등록된 리모트 저장소를 확인  
    : -v 옵션을 사용하면 단축 이름과 URL을 확인 가능
-   git congit  
    : Git 사용 환경 설정을 확인하고 변경 가능
-   git status  
    : 현재 작업 중인 파일의 상태를 확인  
    : Work space 와 Staging area 의 상태를 확인하기 위해 사용
-   git add < 파일명 >  
    : Work space 의 변경 내용을 Staging area 에 추가  
    : Git 은 커밋하기 전에 인덱스에 먼저 commit 할 파일을 추가  
    : git add --all 또는 git add . 을 사용하면 status 의 변경사항을 모두 스테이징 영역에 추가
-   git commit -m "<커밋 메시지>"  
    : 파일 및 폴더의 추가 또는 변경 사항을 저장소에 기록  
    : Staging area 에 등록되어 있는 파일 상태를 기록
-   git log  
    : 다양한 옵션을 조합하여 원하는 형태의 로그를 출력
-   git push origin < 브렌치명 >  
    : 로컬 저장소에서 남겨놓은 파일 변경 이력을 원격 저장소로 전송  
    : -u 옵션을 사용하면 최초 한 번 저장소명과 브렌치명을 입력 후, 생략 가능
-   git pull < 협업자의 리모트 저장소명> < 협업자의 브렌치명 >  
    : 협업자의 원격저장소에서 변경된 내용을 가져와 병합
-   git reset HEAD^  
    : 가장 최근에 동작한 파일을 이전 상태로 복원  
    : 옵션을 통해 특정 커밋까지 이력을 초기화 가능  
    : 되돌린 버전 이후의 버전들은 히스토리에서 삭제
-   git restore < 파일명 >  
    : commit 되지 않은 로컬 저장소의 변경사항을 취소
-   git switch < 브렌치명 >  
    : 사용할 브렌치를 < 브렌치명 > 으로 전환
-   git revert  
    : 특정 커밋을 취소하는 새로운 커밋을 생성  
    : 되돌린 버전 이후의 버전들의 이력 보존