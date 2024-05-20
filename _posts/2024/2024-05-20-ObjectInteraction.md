---
layer: single
title: "오브젝트 상호작용 구현"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 상호작용

- 플레이어가 오브젝트(아이템, 무기, 코인 등)와 상호작용하는것을 구현해보려합니다.

- 여기서 사용할 방법은 Collider를 이용하는것입니다.

- 먼저 상호작용 할 오브젝트 Weapon(Hammer)를 하나 준비했습니다.

![image](/images/2024/2024-05-20/capture_1.PNG)


## 컴포넌트 추가

- 준비한 오브젝트의 모양에 맞는 Colider를 추가합니다.

- 상호작용 영역은 오브젝트보다 크게 설정하기로했고, Colider의 위치와 사이즈를 적절히 조절합니다.

![image](/images/2024/2024-05-20/capture_2.PNG)

![image](/images/2024/2024-05-20/capture_3.PNG)

- Colider의 isTrigger를 체크해주어야 합니다.

- 마지막으로 오브젝트의 Tag(Weapon)를 만들어 지정해줍니다.

- Edit - Project Settings - Input Manager에서 상호작용(Interaction)키 를 만들어줍니다.

![image](/images/2024/2024-05-20/capture_4.PNG)


## Interaction 구현

- 상호작용은 Trigger 이벤트로 간단하게 구현할 수 있습니다.
- Trigger 이벤트는 3가지 종류가 있습니다. 

1. OnTriggerEnter(Collider other) : Trigger 영역에 들어갈 때
2. OnTriggerStay(Collider other)  : Trigger 영역에 들어와있을 때
3. OnTriggerExit(Collider other)  : Trigger 영역을 빠져나갈 때

- 매개변수 other로 충돌한 Collider의 정보를 가져옵니다.


```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class InteractionTest : MonoBehaviour
{
    // .. 가까이 있는 오브젝트를 담을 변수
    GameObject nearObject;

    // .. 상호작용 키[E]를 눌렀는지 확인할 변수
    bool interactionKeydown;


    void Update()
    {
        GetInput();
        Interaction();
    }

    // .. 사용자 입력 받기
    void GetInput()
    {
        interactionKeydown = Input.GetButtonDown("Interaction");
    }

    void OnTriggerStay(Collider other)
    {
        if(other.tag == "Weapon")
        {
            nearObject = other.gameObject;
            Debug.Log(nearObject.name);
        }
    }

    void OnTriggerExit(Collider other)
    {
        if(other.tag == "Weapon")
        {
            nearObject = null;
        }
    }

    // .. 상호작용 함수
    void Interaction()
    {
        if(interactionKeydown && nearObject.tag == "Weapon")
        {
            Debug.Log("무기를 획득했음");
            Destroy(nearObject);
        }
    }
}

```

- OnTriggerStay 함수는 플레이어가 무기 오브젝트의 콜리이더 영역에 있을 때 nearObject 변수에 해당 무기 오브젝트 정보를 담습니다.

- Interaction 함수에서는 nearObject가 'Weapon' 이고, 상호작용키를 눌렀을 때 무기를 획득했다는 로그와함께 오브젝트를 파괴하여 아이템을 먹은것처럼 표현했습니다.

![image](/images/2024/2024-05-20/capture_5.gif)


## OnCollsion과 OnTrigger의 차이

- OnCollsion과 OnTrigger는 모두 충돌을 감지하는것은 같습니다. 

- OnCollsion 함수도 트리거와 마찬가지로 3가지(Stay, Enter, Exit) 이벤트가 있습니다.

- 다만 Collsion는 Trigger와 다르게 물리적인 연산을 수행하며 충돌을 감지합니다. 

- 이때 주의할 점은 Trigger 사용 시 오브젝트 둘중 하나는 isTrigger 옵션이 체크, Rigidbody를 가지고 있어야 한다는 점입니다.
