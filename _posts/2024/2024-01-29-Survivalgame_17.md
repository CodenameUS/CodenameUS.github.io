---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[16]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 무기 장착 표현

- 플레이어가 무기를 얻었을 때, 손에 무기를 들고있는것을 표현했습니다.
- 아래와같이 플레이어의 방향에따라서 반전되며, OrderinLayer도 적절히 변화시키면서 자연스러운 연출을 표현합니다.
- 왼손에 들고있는 무기는 HandLeft, 오른손에 들고있는 무기는 HandRight로 오브젝트를 생성하여 붙여줬습니다.

![image](/images/2024/2024-01-29/capture_1.gif)



## 무기 배치

- 프로젝트 폴더의 Sprite - props 폴더에있는 weapon0(삽)과 weapon3(총)을 Player에게 적절히 붙여줍니다.

![image](/images/2024/2024-01-29/capture_2.png)


### Hand.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Hand : MonoBehaviour
{
    public bool isLeft;
    public SpriteRenderer spriter;

    SpriteRenderer player;

    // .. 무기를 쥐고있는 위치, 회전각(설정한 값에 따라 다름)
    Vector3 rightPos = new Vector3(0.35f, -0.15f, 0);
    Vector3 rightPosReverse = new Vector3(-0.15f, -0.15f, 0);
    Quaternion leftRot = Quaternion.Euler(0,0,-35);
    Quaternion leftRotReverse = Quaternion.Euler(0, 0, -135);

    void Awake()
    {
        player = GetComponentsInParent<SpriteRenderer>()[1];    
    }

    void LateUpdate()
    {
        bool isReverse = player.flipX;

        if(isLeft)
        {
            // .. 근접 무기
            transform.localRotation = isReverse ? leftRotReverse : leftRot; // .. 플레이어 기준 회전
            spriter.flipY = isReverse;
            spriter.sortingOrder = isReverse ? 4 : 6;
        }
        else
        {
            // .. 원거리 무기
            transform.localPosition = isReverse ? rightPosReverse : rightPos;
            spriter.flipX = isReverse;
            spriter.sortingOrder = isReverse ? 6 : 4;
        }
    }
}

```

- rightPos와 leftRot는 제가 설정한 무기의 위치,회전각입니다.
- 플레이어가 보고있는 방향에 따라 무기의 위치, 회전각을 플레이어기준으로 적절하게 반전시키고 움직여서 자연스러운 연출을 표현했습니다.
- 하지만 이렇게만 하면 게임시작과 동시에 무기를 얻지도않았는데 무기를 장착하고있는걸로 보이게되므로 수정하겠습니다.


## 데이터 추가 및 연동

- HandLeft와 HandRight를 비활성화 시켜줍니다.(게임시작시 무기가 없으므로)
- ItemData.cs에 한줄을 추가합니다.

![image](/images/2024/2024-01-29/capture_3.png)

- 프로젝트 폴더의 Data 폴더에서 Item 데이터에 Sprite를 추가해줍니다.

![image](/images/2024/2024-01-29/capture_4.png)

- HandLeft와 HandRight는 Player의 자식 오브젝트이므로, Player.cs 에서 따로 보관해줍니다.


### Player.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.InputSystem;

public class Player : MonoBehaviour
{
    public Vector2 inputVec;
    public float speed;
    public Scanner scanner;
    public Hand[] hands;

    SpriteRenderer sprite;
    Rigidbody2D rigid;
    Animator anim;

    // ... 시작 시 한번만 실행되는 함수 Awake
    void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
        sprite = GetComponent<SpriteRenderer>();
        anim = GetComponent<Animator>();
        scanner = GetComponent<Scanner>();
        hands = GetComponentsInChildren<Hand>(true);    // .. 비활성화된 오브젝트
    }


    // ... 물리 연산 프레임마다 호출되는 함수 FixedUpdate
    void FixedUpdate()
    {
        Vector2 nextVec = inputVec * speed * Time.fixedDeltaTime;    
        // ... 플레이어의 위치(좌표) 이동
        rigid.MovePosition(rigid.position + nextVec);
    }

    // ... 인풋 받기
    void OnMove(InputValue value)
    {
        inputVec = value.Get<Vector2>();        
    }

    
    void LateUpdate()
    {
        anim.SetFloat("Speed", inputVec.magnitude);

        if (inputVec.x != 0)
        {
            // ... 플레이어가 보는 방향 변경
            sprite.flipX = inputVec.x < 0;
        }
    }
}

```

- hands를 초기화해줍니다. 자식 오브젝트들이므로 GetComponentsInChildren를 사용했고, 게임시작시에는 비활성화되어있는 오브젝트들이기때문에 인자값으로 true를 넣어줍니다. 이렇게하면 비활성화된 오브젝트도 초기화할 수 있습니다.

### Weapon.cs

- Weapon 스크립트에서 무기가 새롭게 생성될 때 호출되는 Init() 함수에서 이 Hand들을 활용해줍니다.
- Init() 함수에 다음 로직을 추가해줍니다.

![image](/images/2024/2024-01-29/capture_3.png)

- Hand를 가져옵니다. Hand는 ItemData.cs에서 열거형으로 근접무기(Melee), 원거리무기(Range)로 되어있습니다.
- ItemData 스크립트에서 열거형 원소에 마우스 커서를 갖다대어보면 Index? 가 붙어있는것을 알 수 있습니다.
- 이것을 정수형으로 캐스팅하여 사용합니다. 
- 이제 무기가 새롭게 생성되었을 때, 무기를 장착하고있음을 표현하기 위해 SetActive를 활성화 해줍니다.

