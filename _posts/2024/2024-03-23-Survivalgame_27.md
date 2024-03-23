---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[27]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 모바일빌드

- 완성된 게임을 내 휴대폰에서 실행시키려면 어떻게 하면되는지를 알아봤습니다.
- 먼저, 실제로 모바일에서 빌드하기전에 유니티의 Simulator 기능을 통해 여러 기기에서 테스트해볼 수 있습니다.

![image](/images/2024/2024-03-23/capture_1.png)

- Simulator 탭으로 바꾸고 저는 제 휴대폰 기종에 맞추어 시뮬레이션해봤습니다.

## UI 위치 재설정

- 시뮬레이션을 해보면 프로젝트를 진행하면서 추가된 여러 UI가 시뮬레이터상에 제대로 위치하지 않음을 확인했습니다.
- UI를 휴대폰에 맞추어 다시 설정해주었습니다.

![image](/images/2024/2024-03-23/capture_2.png)


### 시뮬레이션 해보기

![image](/images/2024/2024-03-23/capture_3.gif)


## 조이스틱 추가

- 모바일로 게임할 때는 키보드입력이 아닌 화면 터치로 조작을 하게됩니다. 이에 맞춰 모바일용 조이스틱을 추가해주었습니다.
- 먼저 패키지 매니저에서 Input System의 On-Screen Controls를 추가해줍니다.
- Canvas - Image 이름은 Joy로 하나 만들었습니다.
- Source Image는 프로젝트 폴더의 UI - Joystick 0으로 하고, 여기에 아까 임포트한 Stick을 추가해줍니다.
- 텍스트는 필요없으므로, Joy를 우클릭하여 Prefab - Unpack Completly 하여 연결을 끊어주고 텍스트를 지워줍니다.

![image](/images/2024/2024-03-23/capture_4.png)


- Source Image는 마찬가지로 Joystick 1으로 하고, On-Screen Stick 컴포넌트의 Movement Range를 알맞게 설정합니다.
- Movement Range는 스틱이 어느정도로 움직일 수 있는지를 설정하는 부분입니다.
- Control Path는 Left Stick[Game Pad]로 되어있는 지 확인합니다.

![image](/images/2024/2024-03-23/capture_5.png)

- 마지막으로 Player - Player Input 컴포넌트의 Auto Switch를 체크해제합니다. Auto Switch는 디바이스에 따라 Player Input 알아서 바꿔주는 기능입니다. 일단 마우스로 조이스틱을 움직여 테스트하기위해 끕니다.
- Default Scheme은 이제 Any가 아니고 Gamepad로 바꾸어줍니다.

## 게임종료버튼 추가

- GameStart 아래에 Button Quit 이름으로 버튼을 하나 생성한 뒤 게임종료 버튼을 만들어 주었습니다.

![image](/images/2024/2024-03-23/capture_6.png)

### GameManager.cs

- 조이스틱은 게임을 플레이할 때만 보이게 바꿨습니다.
- 조이스틱의 Scale을 0 0 0으로 합니다.

```c#
 [Header("# Game Object")]
    public bool isLive;
    public Player player;
    public PoolManager pool;
    public LevelUp uiLevelUp;
    public Result uiResult;
    public GameObject enemyCleaner;     // .. 게임승리 시 남은 적 처리하는 Bullet
    public Transform uiJoy;

    public void GameQuit()
    {
        Application.Quit();
    }


     public void Stop()
    {
        isLive = false;
        // .. 유니티 시간 속도(배율) 조절(기본 1)
        Time.timeScale = 0;

        uiJoy.localScale = Vector3.zero;
    }

    public void Resume()
    {
        isLive = true;
        Time.timeScale = 1;
        uiJoy.localScale = Vector3.one;
    }
```

### 테스트

![image](/images/2024/2024-03-23/capture_7.gif)

- 게임종료버튼은 모바일에서만 작동하고 에디터상에서는 작동하지않습니다. 


## 렌더러와 프레임 설정

- Edit - Project Settings - Quality 항목

![image](/images/2024/2024-03-23/capture_8.png)

### GameManager.cs

```c#
void Awake()
    {
        instance = this;
        Application.targetFrameRate = 60;       // .. 60프레임설정
    }
```

## 포스트 프로세싱(후처리)

- Hierarchy뷰에 Volume - Global Volume을 추가합니다.
- Profile 부분에 New를 눌러 새로운 볼륨 프로파일을 만들 수 있습니다.
- 여기에서 다양한 후처리 기법을 사용할 수 있습니다.
- 저는 따로 추가하지 않았습니다.

## 모바일 빌드하기

- File - Build Settings 항목
- 먼저 Android로 플랫폼을 변경합니다. 

![image](/images/2024/2024-03-23/capture_9.png)

- Player Settings를 눌러 몇가지 설정을 해주어야합니다.

1. Company Name, Product Name : 회사명과 게임 이름입니다.
2. Resolution and Presentation - Landscape Right/Left (가로모드)체크해제합니다.
3. Other Settings - Scripting Backend - IL2CPP(어플리케이션을 64비트로 빌드), Target Architectures - ARM64 체크

- 이제 Build 버튼을 눌러 빌드를 진행합니다.
- Build가 완료되면 apk 파일이 생성되고, 이것을 모바일로 옮겨 설치하면 모바일 빌드가 가능하다.

![image](/images/2024/2024-03-23/capture_11.png)

### 빌드시 몇가지 오류

- 빌드할 때 발생했던 몇가지 오류를 해결한 방법입니다.

1. Undead Survivor 프로젝트 폴더의 Icon - Advanced - Default - Compression 을 None으로 변경합니다.
2. 경로상의 한글 문제 : 경로에 한글이 있으면 빌드에 오류가 발생합니다.
3. Key Store : 개발자가 본인을 증명하는 키를 모아둔 저장소 -> 구글 스토어 업로드시에 필요한 키다.

- Player Settings - publishing settings - KeyStore Manager - Create New - Any Where
- 새로운 Key store를 생성하고 입력해주면 된다. (이것때매 빌드가 안돼서 한참 고생함...)

![image](/images/2024/2024-03-23/capture_10.png)

- 생성한 키는 잊지않게 잘 보관해두어야함.