---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[12]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 카메라설정

- 자연스러운 카메라 움직임을 더하기 위하여 카메라설정을 다듬어보겠습니다.
- 먼저 Main Camera를 선택하고, Pixel Perfect Camera 컴포넌트를 추가합니다.

![image](/images/2023-11-13/capture_3.png)

- 그리고 Window - Package Manager 를 선택하여 Cinemachine 패키지를 인스톨합니다.
- 이제 Hierarchy 뷰에서 Cinemachine 를 추가할 수 있는데, Virtual Camera를 추가하고 이름도 바꿔줍니다.
- 이 오브젝트는 카메라의 감독 역할을 수행합니다.

- Virtual Camera를 선택하고 Follow를 Player로 지정해주면 이제 카메라가 플레이어를 따라다니게 됩니다.
- 그리고 Main Camera를 선택하여 아래 Update Method를 Fixed Update로 바꿔주면 플레이어 움직임이 자연스러워지게됩니다.

# UI 제작

- Hierarchy뷰에서 +버튼을 눌러 UI -> Canvas 를 추가합니다.
- 이 Canvas는 UI를 그리기 위한 도화지라고 생각하면 됩니다.
- Canvas 모드에는 3가지가 있는데 디폴트로 설정된값은 Overlay입니다. 두번째로 Camera가있는데 이는 카메라에 맞춰 UI가 보여지게되고 마지막 World Space는 월드공간에 UI가 배치되는것을 말합니다.

- 간단하게 Text를 게임화면에 나타내보겠습니다. Canvas 아래에 UI -> Legacy -> Text를 추가합니다.


![image](/images/2023-11-13/capture_1.png)

![image](/images/2023-11-13/capture_2.png)

- 다시 Canvas 인스펙터창에 보면 Canvas Scaler 컴포넌트가 있는걸 볼 수 있습니다. 이는 해상도를 조절하는것으로 기본적으로 고정픽셀로 설정되어있습니다. 하지만 이설정값은 사용자의 화면 해상도에따라 UI 크기가 다르게 보여지게되므로 Scale With Screen Size를 선택하여 화면크기에따라 UI크기도 달라지게 설정합니다.

![image](/images/2023-11-13/capture_4.png)

- 이제 Canvas 세팅은 완료되었습니다. Text를 지워주고 스크립트를 작성할준비를 합니다.


## HUD.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HUD : MonoBehaviour
{
    public enum InfoType { Exp, Level, Kill, Time, Health }
    public InfoType type;

    Text myText;
    Slider mySlider;

    void Awake()
    {
        myText = GetComponent<Text>();
        mySlider = GetComponent<Slider>();
    }

    void LateUpdate()
    {
        switch (type)
        {
            case InfoType.Exp:

                break;
            case InfoType.Level:

                break;
            case InfoType.Kill:

                break;
            case InfoType.Time:

                break;
            case InfoType.Health:

                break;
        }    
    }
}

```

- HUD에서 다룰 데이터들을 enum 을 사용해 열거하였습니다. 이것을 사용하기 위하여 type 이라는 변수도 선언하였습니다.
- UI에 Text와 Slider를 사용할 예정이므로 두가지를 초기화해주었습니다.
- 5가지 데이터를 갱신하기위해 LateUpdate 함수에서 각 데이터별로 처리할 수 있도록 switch 문을 사용해 틀을 잡아놨습니다.

### 경험치 게이지바 만들기

- Canvas - UI - Slider를 추가합니다.
- 먼저 Anchor 설정을 합니다. Anchor는 화면상의 배치를 도와주는 것으로 Shift를 누르면 기준점을, Alt를 누르면 너비를 확인할 수 있습니다. Shift + Alt 를 눌러 이 게임에 맞는 위치를 선택해줍니다.

![image](/images/2023-11-13/capture_5.png)


- Interactable 체크해제 -> 사용자가 임의로 수정할 수 없도록
- Transition -> None
- Navigation -> None
- Handle Rect - Delete 하여 삭제
- Value -> 게이지가 차오르는 정도

![image](/images/2023-11-13/capture_6.png)


- 설정 후 Slider 아래의 Handle 오브젝트를 삭제합니다.
- BackGround를 선택하고 Anchor를 최대로, Source Image에 Sprite 폴더의 Back0을 지정합니다.
- Fill Area 아래의 Fill 를 선택하고, Left와 Right를 0으로, Source Image를 Sprite 폴더의 Front0 으로 지정합니다.

![image](/images/2023-11-13/capture_7.png)

### HUD.cs 수정

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HUD : MonoBehaviour
{
    public enum InfoType { Exp, Level, Kill, Time, Health }
    public InfoType type;

    Text myText;
    Slider mySlider;

    void Awake()
    {
        myText = GetComponent<Text>();
        mySlider = GetComponent<Slider>();
    }

    void LateUpdate()
    {
        switch (type)
        {
            case InfoType.Exp:
                // .. 현재경험치 / 최대경험치
                float curExp = GameManager.instance.exp;
                float nextExp = GameManager.instance.nextExp[GameManager.instance.level];
                mySlider.value = curExp / nextExp;
                break;
            case InfoType.Level:

                break;
            case InfoType.Kill:

                break;
            case InfoType.Time:

                break;
            case InfoType.Health:

                break;
        }    
    }
}

```

- 경험치바의 value는 0~1사이의 값을 가지고 값에 따라 게이지가 차오르게됩니다.
- 따라서 현재경험치/최대경험치 값을 가지고 게이지가 차도록 했습니다.

