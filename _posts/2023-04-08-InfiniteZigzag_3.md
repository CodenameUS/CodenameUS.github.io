---
layer: single
title: "[Unity] 간단한 3D 게임 만들기[3]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



현재까지 만든 게임에서는 아직 플레이어 죽음과 게임오버가 없습니다.

이번에는 플레이어 죽음과 게임오버를 만들어볼 예정입니다.

우선, Player 오브젝트를 선택하여 IsKinematic를 해제합니다.

이제부터 플레이어는 중력의 영향을 받게됩니다..



## 게임오버

PlayerControl 스크립트를 수정합니다.

Update()함수에 다음을 추가합니다.

![image](/images/2023-04-08/capture_3.png)



플레이어가 중력을 받고 떨어지면 게임을 끝내는 GameOver 함수를 호출합니다.

이제 GameManager 스크립트를 수정합니다.

```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    public GameObject platformSpawner;
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

        //게임을 시작하면 platformSpawner 동작
        platformSpawner.SetActive(true);
    }


    //게임 오버
    public void GameOver()
    {
        //게임오버시 platformSpawner 동작 멈춤
        platformSpawner.SetActive(false);
    }
}

```

수정하였다면 GameManager 스크립트의 PlatformSpawner 부분에 PlatformSpawner 오브젝트를 연결합니다.

![image](/images/2023-04-08/capture_4.png)



PlatformSpawner를 선택하고, inspector를 체크 해제합니다.

![image](/images/2023-04-08/capture_5.png)



Camera 스크립트를 조금 수정합니다.

Update함수부분에 다음을 추가합니다.


![image](/images/2023-04-08/capture_6.png)




Platform 이라는 스크립트를 하나 생성합니다.

Prefabs 폴더의 PlatformPre 프리팹을 선택하고 Platform스크립트를 연결합니다.



```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Platform : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }

    //플레이어가 지나간뒤 바닥 떨어짐
    private void OnCollisionExit(Collision collision)
    {
        if (collision.gameObject.tag == "Player")
        {
            Invoke("Fall", 0.5f);
        }
    }

    void Fall()
    {
        GetComponent<Rigidbody>().isKinematic = false;
        Destroy(gameObject, 1.0f);
    }
}

```




Player 오브젝트를 선택하고 Tag를 Player로 바꿔줍니다.

![image](/images/2023-04-08/capture_7.png)


게임을 실행해서 플레이어가 지나간 뒤 블럭이 떨어지는지,

그리고 플레이어가 떨어졌을 때 카메라가 멈추는지를 확인합니다.

