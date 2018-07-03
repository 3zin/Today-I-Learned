# Git

파일 관리 시스템

## Commit(커밋)

- 커밋 예정의 파일을 add 명령어를 통해 stage 상태에 올려 놓고(커밋 예정) 변경 사항을 한꺼번에 커밋한다
- 일종의 파일 추적. `git add`는 파일을 새로 추적(Tracked)할 때도 사용하고 수정한 파일을 Staged 상태로 만들 때도 사용한다.`git add` 명령을 실행한 후에 또 파일을 수정하면 `git add` 명령을 다시 실행해서 최신 버전을 Staged 상태로 만들어야 한다:
- 커밋 예정 메모리가 아닌 stage area라는 독립적인 공간에 저장해  e놓아 안정성을 높일 수 있음
- Conflict가 일부 파일에서 발생했을 경우 독립적으로 Commit 가능
- 혹은 같은 feature 내에서도 기능에 따라 독립적으로 Commit하고 싶을 때 스테이징 기능을 사용해서 따로 따로 커밋할 수 있음.



## 브랜치

### 통합 브랜치(Integration Branch)

- 통합 브랜치란 언제든지 배포할 수 있는 버전을 만들 수 있어야 하는 브랜치. 그렇기 때문에 늘 **안정적인 상태**를 유지하는 것이 중요합니다. 여기서 '안정적인 상태'란 현재 작업 중인 소스코드가 모바일에서 동작하는 어플리케이션을 개발하기 위한 것이라면, '그 어플리케이션의 모든 기능이 정상적으로 동작하는 상태'를 의미합니다.
- 일반적으로 저장소를 처음 만들었을 때에 생기는 'master' 브랜치를 통합 브랜치로 사용합니다.

### 토픽 브랜치(Topic Branch)

- 피처 브랜치 (Feature)
- 토픽 브랜치란, 기능 추가나 버그 수정과 같은 단위 작업을 위한 브랜치 입니다. 여러 개의 작업을 동시에 진행할 때에는, 그 수만큼 토픽 브랜치를 생성할 수 있습니다.
- 토픽 브랜치는 보통 통합 브랜치로부터 만들어 내며, 토픽 브랜치에서 특정 작업이 완료되면 다시 통합 브랜치에 병합하는 방식으로 진행됨



## 브랜치 전환

- '체크아웃(checkout)' 명령어를 실행하여 원하는 브랜치로 전환 가능. 체크아웃을 실행하면, 우선 브랜치 안에 있는 마지막 커밋 내용이 작업 트리에 펼쳐집니다. 브랜치가 전환 되었으므로 이 후에 실행한 커밋은 전환한 브랜치에 추가됩니다.
- 'HEAD' 란 현재 사용 중인 브랜치의 선두 부분을 나타내는 이름

### stash(스태시)

- 커밋하지 않은 변경 내용이나 새롭게 추가한 파일이 인덱스와 작업 트리에 남아 있는 채로 다른 브랜치로 전환(checkout)하면, 그 변경 내용은 기존 브랜치가 아닌 전환된 브랜치에서 커밋할 수 있습니다.
- 단, 커밋 가능한 변경 내용 중에 전환된 브랜치에서도 한 차례 변경이 되어 있는 경우에는 체크아웃에 실패할 수 있음.
- 이 경우 이전 브랜치에서 커밋하지 않은 변경 내용을 커밋하거나, stash 를 이용해 일시적으로 변경 내용을 다른 곳에 저장하여 충돌을 피하게 한 뒤 체크아웃을 해야 합니다. stash 를 사용하여 작업 트리와 인덱스 내에서 아직 커밋하지 않은 변경을 일시적으로 저장해 둘 수 있습니다. 이 stash 에 저장된 변경 내용은 나중에 다시 불러와 원래의 브랜치나 다른 브랜치에 커밋할 수 있습니다.



## 브랜치 통합

### Merge

- fast-forward merge
- non fast-forward merge
- 변경 내용의 이력이 모두 그대로 남아 있기 때문에 이력이 복잡해짐.

### Rebase

- master의 위치는 그대로 두고, bug-fix를 앞으로 보낸 후 fast-forward merge로 확인

- 이력은 단순해지지만, 원래의 커밋 이력이 변경됨. 정확한 이력을 남겨야 할 필요가 있을 경우에는 사용하면 안됨.
- 예를 들어, 버그 수정 이후 새로운 기능을 추가하려고 한다면 버그 수정이 끝난 이후, 그 수정을 통합 브랜치에 반영한 그 시점을 rebase를 통해 master의 위치는 그대로 두고 개발을 진행하는 것. 나중에 fast-forward merge하면 됨.



## 원격 저장소 처리

### Push

- 로컬에서 원격 저장소로 밀어넣기. push한 브랜치가 fast-forward 병합될 수 있도록 처리해 주어야 함
- 모두가 공유하는 원격 저장소의 커밋을 임의로 변경하거나 덮어쓰면 안됨 

### Pull

- 원격 저장소의 내용을 로컬에 가져와 병합하기(ex. origin/master to master). Conflict가 없을 경우 fast-forward 병합

### Fetch

- 병합하지 않고 원격 저장소의 내용을 가져오기만 하기. 통합하고 싶을 경우에는 FETCH_HEAD merge 혹은 pull
- 원본을 작성한 개발자는 remote 저장소를 적당히 pull해서 받아오면 된다.

### Fork

- 다른 사람의 원격 저장소 코드를 본인의 원격 저장소에 옮겨오기. 이후 클론하고 풀/푸쉬 해가며 수정을 거친 후, 맨 마지막에 Pull Request로 요청을 보내면 된다.



## 커밋 삭제 및 변경

### --amend

- 같은 브랜치 상의 이전 커밋에 새로운 내용 추가 

### reset

- 더이상 필요 없어진 커밋을 버리고 이전 상태로 돌아감. 커밋만 되돌릴수도, 변경한 인덱스도 같이 버리고 되돌릴수도, 아예 작업 트리를 다 버리고 되돌아갈 수도 있음. (soft, mixed, hard)
- reset 전의 커밋은 'ORIG_HEAD'라는 이름으로 참조할 수 있습니다. 실수로 reset 을 한 경우에는, 'ORIG_HEAD'로 reset 하여 reset 실행 전의 상태로 되돌릴 수 있습니다.

### revert - 

- 특정 커밋의 내용을 지우는 새로운 커밋(B')을 만들어 보다 안전하게 처리. 안전하게 삭제할 때 

### rebase -i

- 커밋을 다시 쓰거나 통합할 수 있음. push 하기 전에 이전의 커밋 내용을 정리하고, 누락된 파일을 추가하거나, 그룹으로 묶을 수 있는 커밋을 같이 묶는 것.

<br>

![git_ua](/Users/3zin/Documents/Today-I-Learned/project/git_ua.png)

## Git Flow

- Design Pattern of Git
- 병합(merge) 기반의 솔루션

## 메인 브랜치(Main branch)

'master' 브랜치와 'develop' 브랜치, 이 두 종류의 브랜치를 보통 메인 브랜치로 사용합니다.

- **master**
  'master' 브랜치에서는, 배포 가능한 상태만을 관리합니다. 커밋할 때에는 태그를 사용하여 배포 번호를 기록합니다.
- **develop**
  'develop' 브랜치는 앞서 설명한 통합 브랜치의 역할을 하며, 평소에는 이 브랜치를 기반으로 개발을 진행합니다.

## 피처 브랜치(Feature branch)

피처 브랜치는, 앞서 설명한 토픽 브랜치 역할을 담당합니다.

이 브랜치는 새로운 기능 개발 및 버그 수정이 필요할 때에 'develop' 브랜치로부터 분기합니다. 피처 브랜치에서의 작업은 기본적으로 공유할 필요가 없기 때문에, 원격으로는 관리하지 않습니다. 개발이 완료되면 'develop' 브랜치로 병합하여 다른 사람들과 공유합니다.





* 'add'  명령어는 곧 

## 릴리즈 브랜치(Release branch)

릴리즈 브랜치에서는 버그를 수정하거나 새로운 기능을 포함한 상태로 모든 기능이 정상적으로 동작하는지 확인합니다. 릴리즈 브랜치의 이름은 관례적으로 브랜치 이름 앞에 'release-' 를 붙입니다. 이 때, 다음 번 릴리즈를 위한 개발 작업은 'develop' 브랜치 에서 계속 진행해 나가면 됩니다.

릴리즈 브랜치에서는 릴리즈를 위한 최종적인 버그 수정 등의 개발을 수행합니다. 모든 준비를 마치고 배포 가능한 상태가 되면 'master' 브랜치로 병합시키고, 병합한 커밋에 릴리즈 번호 태그를 추가합니다.

릴리즈 브랜치에서 기능을 점검하며 발견한 버그 수정 사항은 'develop' 브랜치에도 적용해 주어야 합니다. 그러므로 배포 완료 후 'develop' 브랜치에 대해서도 병합 작업을 수행합니다.

## 핫픽스 브랜치(Hotfix branch)

배포한 버전에 긴급하게 수정을 해야 할 필요가 있을 경우, 'master' 브랜치에서 분기하는 브랜치입니다. 관례적으로 브랜치 이름 앞에 'hotfix-'를 붙입니다.

예를 들어 'develop' 브랜치에서 개발을 한창 진행하고 있는 도중에 이전에 배포한 소스코드에 아주 큰 버그가 발견되는 경우를 생각해 보세요. 문제가 되는 부분을 빠르게 수정해서 안정적으로 다시 배포해야 하는 상황입니다. 'develop' 브랜치에서 문제가 되는 부분을 수정하여 배포 가능한 버전을 만들기에는 시간도 많이 소요되고 안정성을 보장하기도 어렵습니다. 그렇기 때문에 바로 배포가 가능한 'master' 브랜치에서 직접 브랜치를 만들어 필요한 부분 만을 수정한 후 다시 'master'브랜치에 병합하여 이를 배포하려고 하는 것이죠.

이 때 만든 핫픽스 브랜치에서의 변경 사항은 'develop' 브랜치에도 병합하여 문제가 되는 부분을 처리해 주어야 합니다.

![git-flow_overall_graph](/Users/3zin/Documents/Today-I-Learned/project/git-flow_overall_graph.png)



- 처음에는 master와 develop 브랜치가 존재합니다. 물론 develop 브랜치는 master에서부터 시작된 브랜치입니다. develop 브랜치에서는 상시로 버그를 수정한 커밋들이 추가됩니다. 
- 새로운 기능 추가 작업이 있는 경우 develop 브랜치에서 feature 브랜치를 생성합니다. feature 브랜치는 언제나 develop 브랜치에서부터 시작하게 됩니다. 기능 추가 작업이 완료되었다면 feature 브랜치는 develop 브랜치로 merge 됩니다. 
- develop에 이번 버전에 포함되는 모든 기능이 merge 되었다면 QA를 하기 위해 develop 브랜치에서부터 release 브랜치를 생성합니다. QA를 진행하면서 발생한 버그들은 release 브랜치에 수정됩니다(동시에 연속적으로 develop 브랜치에 back-merge할 수도 있습니다). QA를 무사히 통과했다면 release 브랜치를 master와 develop 브랜치로 merge 합니다. 마지막으로 출시된 master 브랜치에서 버전 태그를 추가합니다.

- git flow는 크게 Start / Finish의 두 단계로 진행됨 

- feature finish 명령을 입력하면

  1. git flow는 develop branch로 checkout 한 후,
  2. feature branch의 변경 내용을 자동으로 develop branch에 merge하고,
  3. 작업이 끝난 feature branch를 삭제한다.

- release finish를 요청하면 세부적으로는 다음과 같은 작업을 처리한다.

  1. release 브랜치의 코드를 master branch에 merge
  2. release의 이름으로 태그 등록
  3. 릴리즈를 develop branch로 재병합(back-merge)
  4. release branch를 삭제

  

## Reference

우아한 형제들: http://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html

Git 입문용: 누구나 쉽게 이해할 수 있는 Git 입문(https://backlog.com/git-tutorial/kr/)
Git 입문용: Git 간편안내서(https://rogerdudler.github.io/git-guide/index.ko.html)
Git 중급용: 지옥에서 온 Git(https://opentutorials.org/course/2708)
Git 완벽가이드: Pro Git(https://git-scm.com/book/ko/v2)
Git-flow cheatsheet(http://dominhhai.github.io/git-flow-cheatsheet/index.ko_KR.html)
Gitlab 사이트(https://gitlab.com)
Gitlab-ci 사이트(https://about.gitlab.com/features/gitlab-ci-cd/)
유의적 버전(https://semver.org/lang/ko/)

add, modified, tracker, staged (https://git-scm.com/book/ko/v1/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%88%98%EC%A0%95%ED%95%98%EA%B3%A0-%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)