---
layer: single
title: "Github로 Unity 프로젝트 관리하기"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Unity Project 관리

- 그동안 만들었던 게임들을 내 Github에 업로드하여 다른사람과 공유하고, 쉽게 형상관리를 할 수 있게끔 해보려고 찾아봤습니다.
- 제가 직접 해보는 과정들과 발생했던 문제에 대한 해결책을 적어봤습니다.
- 생략된 부분이 많을 수 있습니다.


## Git 및 Github Desktop 설치

- Git 설치 및 설정방법은 인터넷에 수많은 정보글들이 있고, 저는 예전에 세팅을 해놨기때문에 따로 설명하지 않겠습니다.


## Github Repository 생성하기

- 먼저, 프로젝트를 저장할 레포지토리를 생성합니다. 깃헙에서 직접 만들수도있지만 Github Desktop에서 쉽게 할 수 있습니다.
- 좌측상단에 Current repository를 눌러보면 Add 버튼이 있습니다.

![image](/images/2024/2024-04-01/capture_1.png)

- Add 버튼을 눌러서 Create new repository를 선택합니다.

![image](/images/2024/2024-04-01/capture_2.png)

1. Name : 생성할 레포지토리 이름입니다.
2. Description : 레포지토리 설명란입니다.
3. Local path : 내 컴퓨터의 로컬경로입니다. 본인이 관리하기 쉬운곳에 설정하면됩니다.
4. Git ignore : 프로젝트에 원하지 않는 파일들을 Git에서 제외할 수 있는 설정입니다. Unity 항목이 따로있으니 선택합니다.

![image](/images/2024/2024-04-01/capture_3.png)

- Publish Repository를 누르면 레포지토리 생성 끝입니다.
- Keep this code private 옵션을 선택하면 초대받은 사람만 코드를 볼 수 있도록 설정할 수 있습니다.

### Unity Project 설정

- Unity에서 프리팹이나 씬같은 파일을 저장하는 방법은 Binary, Text, 두가지를 섞어 사용하는 혼합방식이 있습니다.
- 개발자에 따라 그 방법이 다를 수 있는데, 깃헙에서는 Binary 형식의 파일을 관리하기 어려우므로 Text 방식으로 바꿔주어야 합니다.

![image](/images/2024/2024-04-01/capture_4.png)

- 먼저 유니티 프로젝트를 열어 [Edit -> Project Settings -> Editor]에서 Asset Serialization(에셋 직렬화) Mode를 Force Text로 바꿔줍니다.(유니티 최신버전에서는 기본값으로 설정되어있음)

## Repository에 Unity 프로젝트 넣기

- 레포지토리를 생성할 때 설정했던 Local path로 내 Unity Project안의 파일을 복사하여 넣어줍니다.

![image](/images/2024/2024-04-01/capture_5.png)

- 그 다음, Github Desktop을 열어보면 Changes 목록에 내가 복붙한 프로젝트 파일들을 볼 수 있고 아래에서 Commit 할 수 있습니다.
- Commit to main 버튼을 누른 후 깃헙에 Push 하면 내 깃허브에 유니티 프로젝트파일이 올라간 것을 확인할 수 있습니다.

## 유의점 및 문제해결

- 해보면서 발생했던 문제들과, 유의점들을 정리해봤습니다.

### CRLF 개행 문자 차이로 인한 문제

![image](/images/2024/2024-04-01/capture_6.png)

- 프로젝트 파일을 Commit 하려하니 위와같은 경고가 발생했습니다.
- 해당 경고문은 OS 따라 달라지는 CRLF 개행 문자 차이로 발생하는 문제로, git 설정을 해주면 해결됩니다.

![image](/images/2024/2024-04-01/capture_7.png)

- [참조링크](https://www.lesstif.com/gitbook/git-crlf-20776404.html)

![image](/images/2024/2024-04-01/capture_8.png)

- 위 링크의 방법대로 하면 문제가 해결됩니다.

### 대용량 깃푸쉬

![image](/images/2024/2024-04-01/capture_9.png)

- 깃허브는 한번 Push 할 때 최대 100MB를 초과하면 안됩니다. 
- 100MB 이상의 파일을 업로드해야할 때는 Git LFS를 사용해야합니다. 
- [참조링크](https://cheonsong.tistory.com/12)

### README 작성하기

- Github에 레포지토리를 생성하고, 파일을 업로드하여 다른사람들과 공유하려할 때 꼭 있으면 좋은것이 README 파일입니다.
- README 파일은 프로젝트에대한 소개와 정보, 설치 및 실행하기위한 방법등을 서술하는곳이므로 다른사람들이 알아보기쉽게 작성하는것이 좋습니다.
- [참조링크](https://velog.io/@luna7182/%EB%B0%B1%EC%97%94%EB%93%9C-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-README-%EC%93%B0%EB%8A%94-%EB%B2%95) 