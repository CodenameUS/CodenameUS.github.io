---
layer: single
title: "[Unity] 간단한 3D 게임 만들기[5]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


마지막으로 메인화면 및 점수를 표시해보겠습니다.


## Score 표시

우선, Hierarchy 뷰에서 UI -> Text를 생성합니다.

Canvas가 생성된 것을 확인할 수 있습니다.

Canvas를 선택하고, UI Scale Mode -> Scale With Screen Size

Refrece Resolution 1080 1920, Match 0.5로 설정해줍니다.

![image](/images/2023/2023-05-08/capture_4.png)




Canvas의 자식으로 있는 Text의 이름을 Score로 바꾸고 Score를 선택하여

Center를 클릭한 다음 아래 처럼 top 중앙에 오도록 합니다.

![image](/images/2023/2023-05-08/capture_5.png)



그리고 Canvas에 Create Empty하여 GameObject를 생성한 뒤 이름을 GameUI로 바꾸고 Score를 자식으로 놓습니다.

![image](/images/2023/2023-05-08/capture_6.png)


GameManager 스크립트를 수정합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class GameManager : MonoBehaviour
{
    public static GameManager instance;

    public GameObject platformSpawner;
    public GameObject Retry;
    public GameObject gameUI;
    public GameObject MainUI;

    public Text scoreText;
    public Text mainText;
    public bool gameStarted;        //게임이 시작됐는지 확인할 변수

    int score = 0;
    float time;

    private void Awake()
    {
        if(instance == null)
        {
            instance = this;
        }
        else if(instance != null)
        {
            Destroy(gameObject);
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

        //게임 시작시 스코어보이기
        gameUI.SetActive(true);
        StartCoroutine(UpdateScore());
    }


    //게임 오버
    public void GameOver()
    {
        //게임오버시 platformSpawner 동작 멈춤
        platformSpawner.SetActive(false);

        //게임 화면 멈추기
        Time.timeScale = 0;

        //1초뒤에 ReloadGame 함수에 전달
        Invoke("ReloadGame", 1);
    }

    //게임재시작
    void ReloadGame()
    {
        Time.timeScale = 1;
        SceneManager.LoadScene("InfiniteZigzag");
    }

    IEnumerator UpdateScore()
    {
        //1초 마다 1점씩 올라감
        while(true)
        {
            yield return new WaitForSeconds(1.0f);
            score++;

            scoreText.text = score.ToString();
        }
    }
}

```

게임시작하면 스코어가 표시되는지, 점수가 잘 오르는지 확인합니다.


