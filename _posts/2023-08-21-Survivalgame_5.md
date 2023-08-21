---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[4]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 애니메이션

- 이번 포스팅에서는 플레이어의 애니메이션을 설정해보겠습니다.

## 플레이어가 보는 방향 전환

- 지금은 좌우로 움직여도 한방향만 바라보고 있습니다.

### Player.cs

- Player 스크립트에 다음 코드를 추가해줍니다.

- filpX 기능은 SpriteRenderer에 있으므로, 변수 선언과 초기화를 해주어야 합니다.

```c#
void LateUpdate()
    {
        if (inputVec.x != 0)
        {
            //플레이어가 보는 방향 변경
            sprite.flipX = inputVec.x < 0;
        }
    }
```

- LateUpdate() 함수는 프레임이 종료 되기 전(다음프레임으로 넘어가기 전) 실행되는 생명주기 함수입니다.

- inputVec.x 값은 좌(-1) 우(1) 가만히(0) 같을 갖고있습니다. 

- 따라서 위의 조건문의 뜻은 inpuVec.x가 0이 아닐 때, 즉 좌우로 움직임이 있을 때 실행됩니다.

- flipX는 말그대로 X방향을 flip 한다는 말로, true false값을 가지고 있습니다.


## 애니메이션 설정

- 이전에도 다루었지만, 유니티에서 애니메이션 설정은 간단합니다.

- Sprite 폴더에 있는 Player0의 Run0 부터 Run5 까지를 선택한 다음 플레이어 오브젝트에 붙여넣어줍니다.

- 그러면 파일을 저장할 폴더를 선택하는 창이 나오는데, Animation 폴더에 저는 Player0_Run.anim 의 이름으로 저장했습니다.

- 마찬가지로 Stand, Dead 애니메이션도 추가해줍니다.

- Player 오브젝트를 눌러보면 인스펙터창에 Animator 컴포넌트가 추가된 것을 확인할 수 있고, Animation 폴더에 플레이어의 애니메이션들이 추가되어 있는것을 볼 수 있습니다.

- 더블클릭해보면 애니메이터 창을 확인할 수 있습니다. 

![image](/images/2023-08-21/capture_1.png)



- 애니메이션 이름들을 단순하게 Stand, Dead, Run 으로 바꾸어주었고, 기본상태는 Stand 이므로 우클릭하여 기본 상태를 Stand로 바꿔줍니다.

- Stand와 Run 애니메이션은 계속 이어지도록, Make Transition 하여 이어주도록 합니다.

- Dead는 Stand, Run 상태와 상관없이 바뀔 수 있으므로, AnyState와 이어줍니다.


![image](/images/2023-08-21/capture_2.png)


- 이제 특정 조건이 되었을 때 애니메이션이 바뀌도록 하기위하여 Parameters를 추가합니다.

1. +버튼을 눌러 Float 형식의 파라미터를 추가합니다. 이름은 Speed

2. +버튼을 눌러 Trigger 형식의 파라미터를 추가합니다. 이름은 Dead

- Stand와 Run 사이의 트랜지션을 선택하고, Stand -> Run 트랜지션에서, Conditions 부분에 Speed를 추가, Greater 0.01로 설정해줍니다.
이것은 Speed가 0.01 보다 클 때 Run 애니메이션으로 바뀌는 조건을 설정해주는 것입니다. 윗부분의 Settings 에서는 Has Exit Time을 체크해제, Transition Duration을 0.01 정도로 설정해줍니다.

- Run -> Stand 도 마찬가지로 Conditions 부분만 Less로, 나머지는 똑같이 설정해주고, Dead 애니메이션에는 Dead 컨디션을 설정해줍니다.

![image](/images/2023-08-21/capture_3.png)

- 마지막으로 애니메이션 설정을 위한 코드작성을 해줍니다.


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

    SpriteRenderer sprite;
    Rigidbody2D rigid;
    Animator anim;

    //시작 시 한번만 실행되는 함수 Awake
    private void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
        sprite = GetComponent<SpriteRenderer>();
        anim = GetComponent<Animator>();
    }


    //물리 연산 프레임마다 호출되는 함수 FixedUpdate
    private void FixedUpdate()
    {
        Vector2 nextVec = inputVec * speed * Time.fixedDeltaTime;    
        //플레이어의 위치 이동
        rigid.MovePosition(rigid.position + nextVec);
    }

    //인풋 받기
    void OnMove(InputValue value)
    {
        inputVec = value.Get<Vector2>();        
    }

    
    void LateUpdate()
    {
        anim.SetFloat("Speed", inputVec.magnitude);

        if (inputVec.x != 0)
        {
            //플레이어가 보는 방향 변경
            sprite.flipX = inputVec.x < 0;
        }
    }
}

```

- 애니메이터 변수를 초기화 해준 뒤 LateUpdate() 함수에서 애니메이션을 설정해줍니다.

- 현재 죽음에 관한 코드는 없기때문에 Dead 애니메이션은 설정하지 않습니다.


## 여러 애니메이션 다루기

- 여기까지 한것은 Player0에 대한 애니메이션 설정이었습니다. 하지만 Sprite 폴더에 보면 여러 캐릭터들이 있습니다. 

- 유니티에서는 여러 애니메이터들을 따로 설정할 필요없이 사용할 수 있도록 해주는 기능이 있습니다.

- 먼저, Player1, Player2, Player3 에 대한 애니메이션을 Player 오브젝트에 붙여 넣어 만들어 줍니다.

![image](/images/2023-08-21/capture_4.png)

- Player 오브젝트를 눌러 Animator를 보면 추가한 다른 애니메이션들이 있을텐데, Delete를 눌러 없애줍니다.

- Animation 폴더에서 + 버튼을 눌러 Animator Override Controller를 추가합니다.

- 이름은 AcPlayer1으로 하고(폴더안에서 파일이 앞쪽에 위치하도록 하기위해) 인스펙터창의 Controller 부분에 Player Animator Controller를 붙여넣어 줍니다. 그다음 밑에서 오버라이드할 애니메이션을 선택해주면 끝입니다.


![image](/images/2023-08-21/capture_5.png)


- 사용하는 방법은, Player 오브젝트를 복사 붙여넣기하여 새로운 오브젝트를 생성해준 뒤, Animator 부분의 Controller에 붙여넣어주기만 하면 됩니다.

![image](/images/2023-08-21/capture_6.png)


- 이제 게임을 실행해보면,

![image](/images/2023-08-21/capture_7.gif)
