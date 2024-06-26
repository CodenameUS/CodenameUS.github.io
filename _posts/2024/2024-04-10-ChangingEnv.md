---
layer: single
title: "작업환경 바꾸고나서 문제"
categories: Unity
tag: [Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---




# 유니티 재설치

- 매번 노트북으로 작업하다보니 불편한 요소가 많아서 데스크톱으로 작업환경을 바꾸기로 결심했다.
- 사실 유니티 설치하는것은 아무런 문제될것이 없었다. 그런데 유니티 에디터 버전이 문제였다.
- 이전까지 내가 만들었던 게임들의 유니티 버전은 2021.3.15f 버전인데, 유니티 허브에는 없었고 홈페이지에 가서 버전을 따로 설치해야됐다.

[유니티 다른버전 에디터 설치](https://unity.com/releases/editor/archive)

![image](/images/2024/2024-04-10/capture_1.PNG)

- 에디터 설치를 끝내고 프로젝트를 열어 모바일빌드를 하려는순간 Platform에서 Android를 선택할 수 없었다.

![image](/images/2024/2024-04-10/capture_2.PNG)

- 유니티 허브에서 안드로이드 SDK 모듈을 추가하려했으나, 왜인지 모르겠지만 모듈추가가 안뜬다...

![image](/images/2024/2024-04-10/capture_3.PNG)


## 유니티 개별요소 설치

- 이곳저곳 블로그 글들을 찾아보니, 유니티허브가아닌 홈페이지에서 다운받았을 경우에는 JDK, SDK, NDK 등 개별 요소들을 따로 설치해야한다는것 같았다.

- 그냥 유니티허브에서 비슷한 에디터버전으로 설치하면 되지않나싶어서 찾아봤는데, 유니티는 프로젝트의 버전을 강제로 업그레이드나 다운그레이드를 해버리면 안좋은결과가 있을것이라고해서 차근차근 설치해보기로했다.


### 안드로이드 빌드를 위한 준비

- 먼저 해당 에디터의 릴리즈노트를 보면 여러 개별요소를 다운로드 할 수 있다.

![image](/images/2024/2024-04-10/capture_4.PNG)

- Android Build Support를 다운받아서 설치했다. 문제는 여기서발생했다. 
- 그냥 설치하기만 하면 되는줄 알았는데, 달라지는것이없었다.
- Unity - Preferences - External Tools를 보면 안드로이드 빌드에 필요한 JDK, SDK, NDK 설치 경로가 지정되어있는데 

![image](/images/2024/2024-04-10/capture_5.PNG)

- 나는 전부 없다고 뜬다. 그냥 전부 설치하기로했다.

#### SDK, NDK, JDK 설치

![image](/images/2024/2024-04-10/capture_6.PNG)

- 2021.3 LTS에 맞는 SDK, JDK, NDK를 설치해주어야한다.
- 먼저 SDK와 NDK 설치를 위해 안드로이드 스튜디오를 설치했다.

![image](/images/2024/2024-04-10/capture_7.PNG)

- More Actions를 눌러 SDK Manager를 열어서 필요한 Tool를 설치했다.

1. Android SDK Build Tools - 30.0.2
2. NDK - 21.3.6528147

![image](/images/2024/2024-04-10/capture_8.PNG)

- 그리고 경로를 복사해두었다가 Unity SDK, NDK 경로에 넣어준다.
- 마지막으로 JDK 설치를 했다. 아래 사이트에서 필요한 버전을 선택하여 설치해주면된다.

[JDK 설치](https://jdk.java.net/java-se-ri/11)


- 설치했던 SDK, NDK, JDK경로를 각 항목에 입력해주면된다.

![image](/images/2024/2024-04-10/capture_9.PNG)


- 경고문으로 You are not using....이라고 뜨는데 무시해도된다. 
- 짧게 요약해서 작성했지만 실제로는 반나절가까이 걸린것같다.. 앞으로 유니티버전은 유니티허브에서 추천하는 버전으로 써야 좀 편할것같다.