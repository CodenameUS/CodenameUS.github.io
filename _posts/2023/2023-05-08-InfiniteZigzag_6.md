---
layer: single
title: "[Unity] 간단한 3D 게임 만들기[6]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



이제 마지막으로 게임같아 보이기위한 메인화면 및 재시작을 조금 수정해보겠습니다.


## 메인화면

Canvas에서 UI->Text를 생성해주고 이름을 Main 으로 바꿉니다.

그 다음 아래와 같이 설정해줍니다.

![image](/images/2023-05-08/capture_a.png)





Text를 하나 더 만들어 이름을 mainText로 바꾸고 Main의 자식으로 놓습니다.

마찬가지로 아래와 같이 설정해줍니다.

![image](/images/2023-05-08/capture_b.png)




GameManager 스크립트를 수정해줍니다.

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
        startAnim();
    }

    //게임 시작
    public void GameStart()
    {
        gameStarted = true;
        //게임시작하면 메인화면 숨기기
        MainUI.SetActive(false);

        //게임시작하면 platformSpawner 동작
        platformSpawner.SetActive(true);

        //게임시작하면 스코어보이기
        gameUI.SetActive(true);
        StartCoroutine(UpdateScore());
    }


    //게임 오버
    public void GameOver()
    {
        //게임오버시 platformSpawner 동작 멈춤
        platformSpawner.SetActive(false);

        //게임화면멈추기
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
        //1초에 1점씩 점수 오름
        while(true)
        {
            yield return new WaitForSeconds(1.0f);
            score++;

            scoreText.text = score.ToString();
        }
    }

    //깜빡이는 애니메이션
    public void startAnim()
    {
        if (time < 0.5f)
        {
            mainText.color = new Color(1, 1, 1, 1 - time);
        }
        else
        {
            mainText.color = new Color(1, 1, 1, time);
            if (time > 1f)
            {
                time = 0;
            }
        }

        time += Time.deltaTime;
    }
}

```


메인화면의 생동감을 표현하기 위해서 "Tap to START" 부분을 깜빡이게 만들었습니다.

Hierarchy뷰의 GameManager를 선택하고, MainUI에 Main을, Main Text에 mainText를 연결해줍니다.

게임을 실행해서 메인화면이 잘 동작하는지, 게임시작시 메인화면이 숨겨지는지 확인합니다.


## 재시작수정

재시작기능을 조금 수정해보겠습니다.

키보드 "R"키를 누르면 게임을 재시작하도록 바꿔보겠습니다.

우선, Canvas에 Text를 하나 추가하고 Retry로 이름을 바꿔줍니다.


![image](/images/2023-05-08/capture_c.png)




게임오버 시 재시작하라는 문구가 뜨도록 해보겠습니다.

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
        startAnim();
    }

    //게임 시작
    public void GameStart()
    {
        gameStarted = true;
        //게임시작하면 메인화면 숨기기
        MainUI.SetActive(false);

        //게임시작하면 platformSpawner 동작
        platformSpawner.SetActive(true);

        //게임시작하면 스코어보이기
        gameUI.SetActive(true);
        StartCoroutine(UpdateScore());
    }


    //게임 오버
    public void GameOver()
    {
        //게임오버시 platformSpawner 동작 멈춤
        platformSpawner.SetActive(false);

        //게임화면멈추기
        Time.timeScale = 0;

        Retry.SetActive(true);
        if (Input.GetKeyDown(KeyCode.R))
        {
            ReloadGame();
        }
    }

    //게임재시작
    void ReloadGame()
    {
        Retry.SetActive(false);
        Time.timeScale = 1;
        SceneManager.LoadScene("InfiniteZigzag");
    }

    IEnumerator UpdateScore()
    {
        //1초에 1점씩 점수 오름
        while(true)
        {
            yield return new WaitForSeconds(1.0f);
            score++;

            scoreText.text = score.ToString();
        }
    }

    //깜빡이는 애니메이션
    public void startAnim()
    {
        if (time < 0.5f)
        {
            mainText.color = new Color(1, 1, 1, 1 - time);
        }
        else
        {
            mainText.color = new Color(1, 1, 1, time);
            if (time > 1f)
            {
                time = 0;
            }
        }

        time += Time.deltaTime;
    }
}


```

Hierarchy뷰의 GamaManager를 선택하여 Retry에 Retry를 연결해줍니다.

게임오버시 Retry문구가 뜨는지 확인합니다.


## 배경음악

배경음악이 없으니 심심한것같아 배경음악을 넣어봤습니다.

여러 방법이 있지만 간단하게 추가해보았습니다.

Main Camera를 선택하고 Audio Source를 추가합니다.

AudioClip부분에 원하는 mp3 파일을 넣습니다.

![image](/images/2023-05-08/capture_d.png)



이렇게 간단한 자동차게임을 만들어봤습니다.

<iframe width="632" height="360" src="https://www.youtube.com/embed/BOv-G-YtSoA" title="게임플레이영상" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>