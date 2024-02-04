---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[19]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 게임시작과 종료

## 게임 시작 및 UI

- 플레이어가 버튼을 누르면 게임이 시작되도록 UI를 만들어보겠습니다.
- Canvas - Create Empty(Game Start) - Image(Title), Button(Button Start)을 추가합니다.
- Image, Button Sprite는 Sprites 폴더에 따로 있으니 추가해줍니다.

![image](/images/2024/2024-02-04/capture_1.png)


- Title과 Button의 위치와 속성등을 적절히 배치합니다.
- 게임시작 UI는 게임시작버튼을 눌렀을 때 사라져야 되므로, Button의 OnClick 이벤트에서 처리합니다.
- Button Start의 OnClick에 GameStart를 추가하고, SetActive를 추가합니다.
- 그다음 게임시작을 누르기 전에는 게임 내 UI들(LevelUp Bar, Hp, Time, Kills... 등등)이 보이지 않았으면하므로, Canvas - Create Empty 하여 HUD라는 오브젝트를 만든 후, Exp, Level, Kill, Timer, Hp 오브젝트를 HUD의 자식으로 넣어줍니다.
- 이번에는 Button Start의 OnClick에 HUD를 추가하고, SetActive를 추가합니다. 그리고 HUD는 비활성화 해줍니다.

![image](/images/2024/2024-02-04/capture_2.png)


- 게임 실행을 해보면, 게임 시작 버튼을 눌러도 게임의 시간이 흐르지않는것을 볼 수 있습니다.
- GameManager 스크립트를 조금 수정하여 버튼을 누르면 게임시간이 흐르도록 수정하겠습니다.

### GameManager.cs

```c#
 public void GameStart()
    {
        health = maxHealth;

        // .. test
        uiLevelUp.Select(0);
       
        isLive = true;

    }
```

- Start() 함수는 이제 public의 GameStart()가 되어 Button Start의 OnClick 이벤트에서 사용할 수 있도록 변경했습니다.
- Button Start의 OnClick에 GameManager를 추가하고, GameStart를 연결합니다.

## 플레이어 피격 구현

- 플레이어가 몬스터에 피격되면 Hp가 깎이도록 만들어봅니다.
- 먼저, GameManager 스크립트에 health와 maxHealth 변수의 타입을 float으로 변경해줍니다.

### Player.cs

```c#
 // .. 플레이어 피격
    void OnCollisionStay2D(Collision2D collision)
    {
        if (!GameManager.instance.isLive)
            return;

        GameManager.instance.health -= Time.deltaTime * 10;

        // .. 플레이어 사망 시
        if(GameManager.instance.health < 0)
        {
            // .. 플레이어의 자식 오브젝트 비활성화
            for(int index = 2;index<transform.childCount;index++)
            {
                transform.GetChild(index).gameObject.SetActive(false);
            }

            anim.SetTrigger("Dead");
        }    
    }
```

- 플레이어 피격은 OnCollisionStay2D 함수에서 구현합니다.
- 플레이어가 몬스터에 피격되면 Hp가 10 깎이는데, Time.deltaTime 을 곱해주어 급사하지않도록 해줍니다.
- 플레이어가 죽으면(Hp가 0 아래) 플레이어가 가진 자식오브젝트들을 순회하면서 비활성화 시켜줍니다.
- 사망하면 플레이어의 애니메이션은 Dead로 변경해줍니다.


## 게임 오버

- GameStart 오브젝트를 재활용하여 게임오버를 만들어보겠습니다. GameStart 오브젝트를 Ctrl+D 하여 복사합니다.
- 이름은 GameResult, Button Start는 Button Retry로 이름을 변경합니다.
- Title의 이미지는 Sprites 폴더의 Title2로, Button의 Text는 돌아가기로 변경합니다.
- Button의 OnClick 이벤트에 있던것들을 모두 제거해줍니다.

### GameManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    // ... 매니저 인스턴스를 어디서든 접근 가능하게 함
    public static GameManager instance;

    [Header("# Game Object")]
    public bool isLive;
    public Player player;
    public PoolManager pool;
    public LevelUp uiLevelUp;
    public GameObject uiResult;

    [Header("# Game Control")]
    // ... 게임 시간과 최대게임시간 변수
    public float gameTime;
    public float maxGameTime = 2 * 10f;

    [Header("# Player Info")]
    // ... 체력, 레벨, 킬수, 경험치, 다음레벨에 필요한 경험치
    public float health;
    public float maxHealth = 100;
    public int level;
    public int kill;
    public int exp;
    public int[] nextExp = { 10, 30, 60, 100, 150, 210, 280, 360, 450, 600 };
    

    void Awake()
    {
        instance = this;
    }

     public void GameStart()
    {
        health = maxHealth;

        // .. test
        uiLevelUp.Select(0);
       
        isLive = true;

    }

    public void GameOver()
    {
        StartCoroutine(GameOverRoutine());
    }

    IEnumerator GameOverRoutine()
    {
        isLive = false;

        // .. Dead Animation을 위한 텀
        yield return new WaitForSeconds(0.5f);

        uiResult.SetActive(true);
        Stop();
    }

    
    public void GameRetry()
    {
        SceneManager.LoadScene(0);
    }

    void Update()
    {
        if (!isLive)
            return;

        gameTime += Time.deltaTime;

        if (gameTime > maxGameTime)
        {
            gameTime = maxGameTime;
        }
    }

    public void GetExp()
    {
        exp++;

        if(exp == nextExp[Mathf.Min(level, nextExp.Length-1)])
        {
            level++;
            exp = 0;
            uiLevelUp.Show();
        }
    }

    public void Stop()
    {
        isLive = false;
        // .. 유니티 시간 속도(배율) 조절(기본 1)
        Time.timeScale = 0;
    }

    public void Resume()
    {
        isLive = true;
        Time.timeScale = 1;
    }
}

```

- GameRetry() 함수에서 처음 화면으로 돌아가도록 했습니다.
- 플레이어가 사망하면 isLive = false로 바꾸어 제어를 못하도록 했고, 플레이어가 사망한 애니메이션을 볼 수 있도록 0.5초 텀을 둔 뒤, GameResult UI를 띄우도록 했습니다.

## 게임승리

- 남은시간동안 생존하여 게임에 승리하는것을 구현하겠습니다.
- 생존하여 게임에서 승리했을 때, 남아있는 몬스터들을 모두 제거하는 연출을 위해 Bullet을 하나 만들어보겠습니다.
- Hierarchy뷰에서 Create Empty - enemyCleaner를 만들어줍니다.

![image](/images/2024/2024-02-04/capture_3.png)


- 게임승리 UI를 만들기 위해 GameResult 오브젝트의 Title를 Title Over로 이름을 변경하고, 복사하여 Title Victory의 오브젝트를 만들어줍니다.
- 게임에서 패배했을 때는 Title Over를, 승리했을 때는 Title Victory UI를 띄워야합니다. 그것을 위한 스크립트 Result를 만들어줍니다.

### Result.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Result : MonoBehaviour
{
    public GameObject[] titles;

    public void Lose()
    {
        // .. 패배 Title
        titles[0].SetActive(true);
    }

    public void Win()
    {
        // .. 승리 Title
        titles[1].SetActive(true);
    }
}

```

- 두가지 title을 담을 변수를두고, 단순히 승리, 패배했을 때 알맞는 UI를 띄우도록 했습니다.

### GameManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    // ... 매니저 인스턴스를 어디서든 접근 가능하게 함
    public static GameManager instance;

    [Header("# Game Object")]
    public bool isLive;
    public Player player;
    public PoolManager pool;
    public LevelUp uiLevelUp;
    public Result uiResult;
    public GameObject enemyCleaner;     // .. 게임승리 시 남은 적 처리하는 Bullet

    [Header("# Game Control")]
    // ... 게임 시간과 최대게임시간 변수
    public float gameTime;
    public float maxGameTime = 2 * 10f;

    [Header("# Player Info")]
    // ... 체력, 레벨, 킬수, 경험치, 다음레벨에 필요한 경험치
    public float health;
    public float maxHealth = 100;
    public int level;
    public int kill;
    public int exp;
    public int[] nextExp = { 10, 30, 60, 100, 150, 210, 280, 360, 450, 600 };
    

    void Awake()
    {
        instance = this;
    }

     public void GameStart()
    {
        health = maxHealth;

        // .. test
        uiLevelUp.Select(0);
        Resume();       
    }

    public void GameOver()
    {
        StartCoroutine(GameOverRoutine());
    }

    IEnumerator GameOverRoutine()
    {
        isLive = false;

        // .. Dead Animation을 위한 텀
        yield return new WaitForSeconds(0.5f);

        uiResult.gameObject.SetActive(true);
        uiResult.Lose();
        Stop();
    }

    public void GameVictory()
    {
        StartCoroutine(GameVictoryRoutine());
    }

    IEnumerator GameVictoryRoutine()
    {
        isLive = false;
        enemyCleaner.SetActive(true);

        // .. Dead Animation을 위한 텀
        yield return new WaitForSeconds(0.5f);

        uiResult.gameObject.SetActive(true);
        uiResult.Win();
        Stop();
    }
    public void GameRetry()
    {
        SceneManager.LoadScene(0);
    }

    void Update()
    {
        if (!isLive)
            return;

        gameTime += Time.deltaTime;

        if (gameTime > maxGameTime)
        {
            gameTime = maxGameTime;
            GameVictory();
        }
    }

    public void GetExp()
    {
        // .. 게임종료시 레벨업 안하도록
        if (!isLive)
            return;

        exp++;

        if(exp == nextExp[Mathf.Min(level, nextExp.Length-1)])
        {
            level++;
            exp = 0;
            uiLevelUp.Show();
        }
    }

    public void Stop()
    {
        isLive = false;
        // .. 유니티 시간 속도(배율) 조절(기본 1)
        Time.timeScale = 0;
    }

    public void Resume()
    {
        isLive = true;
        Time.timeScale = 1;
    }
}

```

- 주요 변경점입니다. 기존에 uiResult 변수 타입이었던 GameObject를 Result로 변경합니다.
- GameOverRoutine()에서 Lose()를 호출합니다.
- GameVictory(), GameVictoryRoutine()을 추가하여 게임승리시 로직을 추가했습니다.
- 게임승리시에는 enemyCleaner를 활성화하여 남아있는 몬스터를 모두 처치하는 연출을 보여줍니다.
- 게임이 끝났음에도 enemyCleaner를 통해 몬스터를 처치했을 때, 레벨업을 하지않도록 GetExp() 함수에 반환로직을 추가합니다.
- 게임오버, 승리 후 돌아가기버튼을 누르면 Stop() 함수에서 timeScale을 0으로 했던것을 GameStart() 함수에서 Resume() 호출을 통해 다시 시간이 흐르도록 했습니다.