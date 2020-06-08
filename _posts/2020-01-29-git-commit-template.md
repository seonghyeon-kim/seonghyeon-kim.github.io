---
layout: post
title: git commit template
tags:
- git
---
### 개요
facebook에서 git commit 메세지에 템플릿을 적용할 수 있다는 글을 보았다. 무언가 깃에 올라간 사항 중에 찾고 싶은게 있을때, 히스토리 하나하나 뒤져가면서 봐야했었다. 이글을 보고 commit template을 적용하면 commit 메세지를 대충 적을 수 없을 뿐더러 굉장히 commit 기록들이 깔끔해진다.

현재의 commit 기록들
![image](./images/gittemplate/1.png)
이러한 기록들이 어떻게 좋아질지 사용해보고 확인해보자.

---
### 적용
적용방법은 간단합니다. 저는 mac 사용자이기에 mac 기준으로 쓴다.
먼저 terminal 하나를 열고
```console
touch ~/.gitmessage.txt
```
git message config를 할 수 있는 텍스트 파일을 하나 만들어 준 후 에디터로 열어준다.
```console
nano ~/.gitmessage.txt
```
원하는 템플릿을 만들어준다. 나는 아래형식으로 만들어주었다. 사실 검색해보니 잘 만들어놓은 template이 있길래 우리 팀에 적용하기 편하게 간소화하여 만들었다.
```console
# <타입>: <title>

## ##### title은 꼭 필요한 정보만 ##########

## # 내용은 위에 작성

# --- COMMIT END ---
# <타입> 리스트
# feat : 기능 (새로운 기능)
# fix : 버그 (버그 수정)
# style : 스타일 (코드 형식, 세미콜론 추가: 비즈니스 로직에 변경 없음)
# docs : 문서 (문서 추가, 수정, 삭제)
# test : 테스트 (테스트 코드 추가, 수정, 삭제: 비즈니스 로직에 변경 없음)
# ------------------
```
이렇게 적은 후 적용해주자.
```console
git config --global commit.template ~/.gitmessage.txt
```
이제 git add 후 git commit 명령어를 치면 터미널에 만든 템플릿이 보일 것이다.
![image](./images/gittemplate/2.png)
이후 template을 팀내부에서 어떻게 사용할지 정하고 사용하면 좋을 것 같다.


