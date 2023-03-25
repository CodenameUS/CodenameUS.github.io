---
layer: single
title: "[Unity] 간단한 3D 게임 만들기[1]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


## 자동차움직이기

자동차를 움직이기 위한 스크립트를 작성해보겠습니다.

우선, Assets 폴더에 새로운 Scripts 폴더를 생성합니다.

Scripts 폴더안에 새로운 PlayerControl 라는 이름의 C# 스크립트를 만들어 Player 오브젝트에 붙여줍니다.

![image](/images/2023-03-25/capture_1.png)



Player 오브젝트를 클릭해 스크립트가 적용되었는지 확인한 뒤 Is Kinematic 를 체크해줍니다.


![image](/images/2023-03-25/capture_2.png)



### PlayerControl.cs

```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerControl : MonoBehaviour
{

    public float moveSpeed;     //이동속도
    bool sideMoving = true;     //방향전환
    bool firstInput = true;     //첫클릭 확인용


    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        if(GameManager.instance.gameStarted)
        {
            Move();
            CheckInput();
        }
    }

    void Move()
    {
        //플레이어 이동
        transform.position -= transform.forward * moveSpeed * Time.deltaTime;
    }

    //게임준비(첫 클릭인지 확인하는함수)
    void CheckInput()
    {
        //첫 클릭이라면 무시
        if(firstInput)
        {
            firstInput = false;
            return;
        }

        //두번째 클릭부터 마우스클릭으로 방향전환
        if(Input.GetMouseButtonDown(0))
        {
            ChangeDirection();
        }
    }

    //플레이어 방향전환
    void ChangeDirection()
    {
        if(sideMoving)
        {
            sideMoving = false;

            transform.rotation = Quaternion.Euler(0, -90, 0);
        }
        else
        {
            sideMoving = true;
            transform.rotation = Quaternion.Euler(0, -180, 0);
        }
    }
}

```


이제 GameManager 라는 이름의 스크립트를 만들어줍니다.

### GameManager.cs

```c++

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager instance;

    public bool gameStarted;        //게임이 시작됐는지 확인할 변수

    private void Awake()
    {
        if(instance == null)
        {
            instance = this;
        }
    }

    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        if(!gameStarted)
        {
            //마우스 클릭시 차가 움직임
            if(Input.GetMouseButtonDown(0))
            {
                GameStart();
            }
        }
    }

    //게임 시작
    public void GameStart()
    {
        gameStarted = true;
    }


    //게임 오버
    public void GameOver()
    {

    }
}

```

GameManager 라는 이름의 오브젝트를 만들고, GameManager.cs를 붙여줍니다.

그 다음, Player 오브젝트를 선택하고 moveSpeed를 8로 설정해줍니다.

![image](/images/2023-03-25/capture_3.png)




![video](/videos/3D%20Infinity%20zigzag%20-%20InfiniteZigzag%20-%20Windows%2C%20Mac%2C%20Linux%20-%20Unity%202021.3.15f1%20Personal_%20_DX11_%202023-03-25%2021-57-42.mp4)
