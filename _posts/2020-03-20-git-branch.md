---
layout: post
title: git branch
tags:
- git
---


### 개요

현재 우리는 개발을 하면서 master에 대고 push하고 서버에 반영하고 있다.
실 운영중인 서버에 바로 반영이 되는 것이 아닌, test 서버에 반영을 하고 문제가 없을 시 git 주소의 관리자가 merge를 하면, 실 운영중인 서버에 반영할 수 있게끔 변경을 하려한다.
검색을 해보니 회사마다 운영방침이 조금씩 다르다.

- master
운영중인 서버
- development
다음 버전을 위한 개발용 branch
- hotfix
긴급히 버그를 수정해야 할때 사용하는 branch
- feature
새로운 기능만을 추가할 때 사용하는 branch

위처럼 모든 branch를 사용했을때 이점들은 분명히 효과적으로 다가올 것이라고 예상이 되지만, 우리 팀의 상황에서 모든것을 다 적용하기엔 조금 시간적, 관리에도 무리가 있을 것이라고 생각이 든다. 그리하여 development branch 하나만 만들어서 적용해보려 한다.

---

### branch 설정하기

local에 branch 생성
```console
​ Administrator@WIN10-20181126 ~ (master) $ git branch <branch-name>
​ Administrator@WIN10-20181126 ~ (master) $ git branch 
* master
<branch-name>
```
branch로 전환
```console
​ Administrator@WIN10-20181126 ~ (master) $ git checkout <branch-name>
​ Administrator@WIN10-20181126 ~ (<branch-name>) $ git branch
* <branch-name>
master
```
최초 push 시
```console
​ Administrator@WIN10-20181126 ~ (<branch-name>) $ git push --set-upstream origin development
```

remote 저장소 branch로 변경
```console
​ Administrator@WIN10-20181126 ~ (<branch-name>) $ git branch --set-upstream-to=origin/<branch-name>
```
이렇게 세팅을 하게 되면 로컬에서 branch를 바라보고 push, pull 작업들을 할 수 있다.
이후 push 하게 되면 github 페이지에서 확인할 수 있다.
빨간 박스 아래 branch 메뉴에서 설정한 branch를 클릭하여 이동하면 해당 branch의 commit 기록들을 볼 수 있다.

---
### 누군가 이미 만들어 놓은 branch 에 작업을 같이하고 싶다면?
```console
​ Administrator@WIN10-20181126 ~ (<master>) $ git remote update
​ Administrator@WIN10-20181126 ~ (<master>) $ git branch -a master remotes/development
```
위처럼 -a 옵션을 주게될 경우 원격저장소의 모든리스트까지 확인할 수 있다.
```console
​ Administrator@WIN10-20181126 ~ (<master>) $ git checkout -t origin/<branch-name>
```
---

### master 에 branch merge 하기

현재 local 환경 확인하기
```console
​ Administrator@WIN10-20181126 ~ (<branch-name>) $ git branch
* <branch-name>
master
```
branch master로 전환
```console
​ Administrator@WIN10-20181126 ~ (<branch-name>) $ git checkout master
Switched to branch ‘master’


Your branch is up to date with ‘origin/master’.
```
branch 바뀌었는지 확인하기
```console
​ Administrator@WIN10-20181126 ~ (master) $ git branch
<branch-name>
* master
```
merge 하기
```console
​ Administrator@WIN10-20181126 ~ (master) $ git merge <branch-name>

----- pull 액션과 함께 어느파일이 바뀌었다 라고 내용이 나온다-----

​ Administrator@WIN10-20181126 ~ (master) $ git push
```
이렇게 push까지 마친다면 branch에서 작업했던 commit 들이 master에 push가 된다.
**github** 최종 관리자가 **branch** 를 확인 후 이상이 없다면 **merge** 를 해주면 된다**.**
