---
layer: single
title: "유니티 이벤트시스템(Event System), 이벤트(Event), 액션(Action)"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

이번 포스팅에서는 유니티에서 UI를 사용할 때 사용자의 입력을 처리하는 핵심적인 구성요소인 Event System에 대해서
정리했습니다.

Event System은 Canvas를 생성하거나, UI > EventSystem을 선택하여 생성할 수 있습니다.

![image](/images/2024/2024-12-10/capture_1.PNG) 

<br>

Event System은 Canvas를 생성하면 자동으로 함께 생성됩니다.


## Canvas

Canvas는 모든 UI 요소들이 속해야하는 영역입니다. 즉 UI를 그리는 도화지 역할입니다.

따라서 Button, Image, Text 등 모든 유니티 UI들은 Canvas 오브젝트 아래에 존재해야하며, 그렇지 않은 UI는 그려지지 않습니다.

Canvas및 UI요소들은 모두 UI용 트랜스폼인 RectTransform을 가지게됩니다.

RectTransform을 통해 화면상의 상대적 위치등을 정할 수 있습니다.

![image](/images/2024/2024-12-10/capture_2.PNG) 


* **Canvas**
    - Canvas 컴포넌트는 UI가 배치되고 그려지는 추상적인 공간으로 모든 UI 요소는 Canvas 컴포넌트가 붙은 GameObject 하위 요소가 되어야합니다.
        - Render Mode는 UI가 렌더링되는 방식을 결정합니다.
    - Canvas 컴포넌트는 캔버스 오브젝트의 핵심입니다.

* **Canvas Scaler**
    - Canvas Scaler는 캔버스에 배치된 모든 UI의 스케일과 픽셀 밀도를 결정합니다.
    - UI Scale Mode 프로퍼티는 캔버스 내의 UI 요소들의 크기를 결정합니다.
    - 화면의 크기, UI 위치, 비율 등의 관계를 정의할 수 있습니다.

* **Graphic Raycaster**
    - Graphic Raycaster는 캔버스에 레이캐스트를 하기 위한 컴포넌트입니다.
    - UI에 광선(Ray)를 쏘아 캔버스에 있는 UI와 충돌했는지 체크합니다.
    - Event System은 이것을 이용하여 이벤트를 감지합니다.


## 유니티 Event System

Event System은 씬 하나에 한개만 생성됩니다.

![image](/images/2024/2024-12-10/capture_3.PNG) 

Event System의 주요 역할은 다음과 같습니다.(Documentation 참고)

- 어떤 게임 오브젝트를 선택할지 관리
- 어떤 입력 모듈을 사용할지 관리
- 레이캐스팅 관리
- 필요에 따라 모든 입력 모듈 업데이트

<br>

* **입력모듈**

입력 모듈은 이벤트 시스템을 어떻게 생생하게 동작하게 할지에 대한 메인로직으로 다음에 사용됩니다.
- 입력처리
- 이벤트 스테이트 관리
- 씬 오브젝트에 이벤트 전송

한 번에 한 입력 모듈만 이벤트 시스템에 활성화할 수 있으며 Event System 컴포넌트로 동일 게임 오브젝트의 컴포넌트여야 합니다.

* **레이캐스터**

레이캐스터는 포인터가 가리키는 것을 계산하는 데 사용되며 일반적으로 입력 모듈이 포인팅 디바이스가 무엇 위에있는지
산출하는 씬과 관련된 레이캐스터를 사용합니다.

다음 3개의 레이캐스터가 기본값으로 제공됩니다.
- 그래픽 레이캐스터 - UI 요소에 사용
- 물리 2D 레이캐스터 - 2D 물리 요소에 사용
- 물리 레이캐스터 - 3D 물리 요소에 사용


<br>

### 유니티 Event System 사용법

이벤트에 대한 Raycast는 Canvas의 Graphic Raycaster 컴포넌트에서 쏴줍니다.

어떤 종류의 이벤트인지에 대한 답을 Raycast로 쏴주는것은 EventSystem 입니다.

이 이벤트에 대한 Raycast를 받기 위해서는 UI들은 Raycast Target 가 체크되어있어야합니다.

```c#
// 네임스페이스 정의
using UnityEngine.EventSystems;

// 함수
if(EventSystem.current.IsPointerOverGameObject())
    return;
```

* **IsPointerOverGameObject**
    * 마우스 클릭 이벤트가 UI 위(EventSystem 오브젝트 위)에서 이루어졌는지 체크하는 함수입니다.
        - 그렇다면 true 아니면 false가 반환됩니다.
    * current : 현재 EventSystem 반환
    * 파라미터로 마우스 포인터 ID가 들어갈 수 있습니다.
        - 좌클릭(-1)
        - 첫 번째 터치(0)


**이벤트 인터페이스 종류**

이벤트 인터페이스를 상속했다면 반드시 해당 인터페이스의 함수를 오버라이딩 해야합니다.

* PointerEventData : 마우스 혹은 터치 입력 이벤트에 관한 정보들이 담겨있습니다.

```c#
// 1. 마우스 클릭 - IPointerClickHandler
public class EventTest : MonoBehaviour, IPointerClickHandler
{
    public void OnPointerClick(PointerEventData eventData)
    {
        // 마우스 클릭시 발생하는 이벤트 함수
    }
}
```

```c#
// 2. 마우스 드래그 - IBeginDragHandler, IDragHandler, IEndDragHandler
public class EventTest : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public void OnBeginDrag(PointerEventData eventData)
    {
        // 이 스크립트가 붙은 오브젝트에 마우스 드래그를 시작했을 때 호출
    }

    public void OnDrag(PointerEventData eventData)
    {
        // 이 스크립트가 붙은 오브젝트에 마우스 드래그 중인 동안 계속 호출
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        // 이 스크립트가 붙은 오브젝트를 마우스 드래그가 끝났을 때 호출
    }
}
```

```c#
// 3. 마우스 드롭 - IDropHandler
public class EventTest : MonoBehaviour, IDropHandler
{
    public void OnDrop(PointerEventData eventData)
    {
        // 이 스크립트가 붙은 오브젝트에 마우스 드롭 이벤트가 발생했을 때 호출
    }
}
```

* OnEndDrag - 나 자신이 드래그가 끝났을 때 호출
* OnDrop - 누군가 나한테 드롭했을 때 호출

이 외에도 여러가지 이벤트 인터페스들이 있습니다.

<br>

## 이벤트란

이벤트(Event)란 클릭이나 터치등에 따른 반응을 처리하는 하나의 시스템입니다.

이벤트에서는 제공자와 구독자가 각각 다음과 같은 역할을 수행합니다.

**제공자** : 사용자의 행동을 기다리다가 구독자에게 알리는 역할

**구독자** : 공급자를 구독해서 사용자의 행동을 전달받아 반응하는 역할

제공자의 여러 이벤트들을 Listener와 같은 구독자들이 구독해서 정보를 기다리고 있는것입니다.

## Unity Event 사용법

UnityEvent의 사용방법입니다.

```c#
// 네임스페이스 정의
using UnityEngine.Events;
```

<br>

유니티 이벤트(제공자)를 정의하는 방법입니다.

```c#
// 이벤트 정의
public UnityEvent OnKeyPressed;
```
<br>

```c#
// 이벤트(제공자)를 호출하는 방법 - 1 : Invoke 함수 사용
void Update()
{
    if(Input.GetKeyDown(KeyCode.Space))
    {
        if(OnKeyPressed != null)
            OnKeyPressed.Invoke();      // 이벤트 핸들러 호출
    }
}

// 이벤트(제공자)를 호출하는 방법 - 2 : ?. 연산자 사용
void Update()
{
    if(Input.GetKeyDown(KeyCode.Space))
    {
        OnKeyPressed?.Invoke();
    }
}
```

```c#
// 다른 클래스(TestingEvents)에서 유니티이벤트 구독
private void Start() {
    TestingEvents testingEvents = GetComponent<TestingEvents>();
    testingEvents.OnKeyPressed.AddListener(Test_OnSpacePressed); //구독
}

//구독자로 쓸 함수
private void Test_OnSpacePressed() 
{
    Debug.Log("스페이스바 누르기");
}
```

일반적인 이벤트들과 다르게 **AddListener** 이라는 콜백함수를 씁니다.

반대로 구독을 취소하기 위해서는 **RemoveListener**을 쓰면됩니다.

<br>

## Unity Action 사용법

기본적으로 유니티 액션은 유니티 이벤트와 같은 역할을 합니다.

유니티 이벤트와의 차이점은
- public으로 해도 인스펙터창에 보이지않음
- 구독과 구독취소는 +=, -= 연산자 사용
- UnityEvent의 AddListener에 넣을 수 있음

```c#
public UnityEvent OnKeyPressed;
public UnityAction unityAction;

void Start()
{
    OnKeyPressed.AddListener(unityAction);
}
```