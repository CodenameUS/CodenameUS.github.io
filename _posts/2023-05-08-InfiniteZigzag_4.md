---
layer: single
title: "[Unity] 간단한 3D 게임 만들기[4]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


## 게임 재시작

GameManager 스크립트를 수정합니다.

네임스페이스 부분에

using UnityEngine.SceneManagement; 를 추가합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

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

        //1초뒤에 ReloadGame 함수에 전달
        Invoke("ReloadGame", 1);
    }

    //게임재시작
    void ReloadGame()
    {
        SceneManager.LoadScene("InfiniteZigzag");
    }
}

```


게임 재시작을 위해 Scene을 추가합니다.

File -> Build Settings -> InfiniteZigzag 씬을 추가합니다.

![image](/images/2023-05-08/capture_1.png)




## 라이트고정하기

게임을 재시작하면 라이트가 꺼지는 현상이 나타나므로

라이트를 고정하려면 라이트맵을 만들어야 합니다.

Window -> Rendering -> Lighting 에 들어갑니다.

Generating Lighting을 눌러 라이트맵을 생성합니다.


![image](/images/2023-05-08/capture_2.png)




라이트맵이 생성되었는지 확인합니다.


![image](/images/2023-05-08/capture_3.png)





이제 게임을 시작하고 게임오버후 1초 후에 재실행되는지, 라이트가 고정되었는지 확인합니다.

