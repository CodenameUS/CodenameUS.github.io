---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[2]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 플레이어 이동 구현

- Scripts 폴더를 하나 생성하고, C# 스크립트(Player)를 하나 생성합니다.

![image](/images/2023-08-18/capture_1.png)

- Player.cs 스크립트를 Player 오브젝트에 붙여줍니다.


## Player.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player : MonoBehaviour
{
    public Vector2 inputVec;
    public float speed;

    Rigidbody2D rigid;

    // ... 시작 시 한번만 실행되는 함수 Awake
    private void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        inputVec.x = Input.GetAxisRaw("Horizontal");       // ... 좌우값
        inputVec.y = Input.GetAxisRaw("Vertical");         // ... 상하값
    }

    // ... 물리 연산 프레임마다 호출되는 함수 FixedUpdate
    private void FixedUpdate()
    {
        Vector2 nextVec = inputVec.normalized * speed * Time.fixedDeltaTime;    
        // ... 플레이어의 위치(좌표) 이동
        rigid.MovePosition(rigid.position + nextVec);
    }
}

```


- Awake() 함수는 게임 실행 시 한번만 실행되는 함수로, 보통 변수의 선언을 여기에서 합니다.

- Update() 함수에서, 플레이어 상하좌우 값을 GetAxisRaw로 받아줍니다.

- Horizontal은 좌우값 (-1~1), Vertical은 상하값 (-1~1)

- FixedUpdate() 함수에서는 물리 로직을 작성합니다.

- nextVec 변수는 inputVec의 값을 normalized한 값(대각선으로 움직일 때 값이 더 크기때문)과 speed(사용자가 따로 설정) 그리고, Time 클래스의 fixedDeltaTime을 곱한값을 저장합니다. 

- Time.fixedDeltaTime은 FixedUpdate() 함수에서 사용할 수 있고, 사용자들의 컴퓨터 프레임이 전부 다르기 때문에 똑같이 맞춰주기 위함입니다.

- 플레이어 이동은 마지막 MovePosition()을 통하여 현재 위치에서 + nextVec 을 한 값으로 위치를 변경함으로써 구현합니다.

- 마지막으로, Hierarchy 뷰의 Player를 선택하고, speed 를 설정해준뒤 실행해보면 캐릭터가 움직이는것을 확인할 수 있습니다.




![image](/images/2023-08-18/capture_2.gif)
