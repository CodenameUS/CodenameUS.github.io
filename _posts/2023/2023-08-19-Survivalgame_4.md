---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[3]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---




# 패키지 설치

- 저번 포스팅에서 Horizontal과 Vertical을 이용하여 플레이어 이동을 구현했습니다. 

- 하지만 이것은 구식방법으로, 유니티에서 제공하는 Input System을 적용해보도록 하겠습니다.


방법은 다음과 같습니다.

1. Window - Pacakage Manager - Packages - Unity Registry - Input system 을 선택하고, Install 합니다.

2. 경고창이 뜨면 Yes를 누르고 유니티를 재시작합니다.


## Input Action 설정하기

- 패키지 인스톨이 완료되면, Hierarchy 뷰의 Player 오브젝트를 선택하고, Player Input 컴포넌트를 추가해줍니다.

![image](/images/2023/2023-08-19/capture_1.png)

- Create Actions... 를 눌러, Undead Survivor 폴더 안에 Player 이름의 프로필 에셋을 생성합니다.

![image](/images/2023/2023-08-19/capture_2.png)

- Move를 열어보면, 키보드 WASD, 조이스틱, 게임패드 등 여러 디바이스들의 인풋을 받을수 있도록 되어있는것을 확인할 수 있습니다.

![image](/images/2023/2023-08-19/capture_3.png)


- 그리고, 오른쪽에서 반환 타입, 인터랙션, 프로세서등을 설정할 수 있습니다. 그중에서 프로세서의 Normalize Vector 2 를 추가합니다.

![image](/images/2023/2023-08-19/capture_4.png)


## Player.cs 스크립트 변경

- 인풋 방식이 바뀌었으니, 스크립트의 내용도 수정합니다.

- Player Input 컴포넌트에 보면 다음과 같은 함수들을 지원하는것을 볼 수 있습니다.

![image](/images/2023/2023-08-19/capture_5.png)



```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.InputSystem;

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


    // ... 물리 연산 프레임마다 호출되는 함수 FixedUpdate
    private void FixedUpdate()
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
}

```

- 네임스페이스에 InputSystem을 사용할 수 있도록 추가해줍니다.

- 기존의 Update() 함수에 있던 내용은 삭제하고, OnMove() 함수에서 새롭게 작성합니다.

- InputValue 라는 타입의 인자를 받아, inputVec 변수에 넣어줍니다.

- 이때, Get은 함수형태로, 꺾새<>와 괄호()가 필요합니다. 꺾새에는 아까 프로필 애셋에서 설정했던 반환값 Vector2를 넣어줍니다.

- FixedUpdate() 함수에 nextVec 변수에 inputVec.normalized 되어있던것도 이제 자동으로 정규화가 되면서 필요가 없어졌으므로 지워줍니다.


![image](/images/2023/2023-08-19/capture_6.gif)
