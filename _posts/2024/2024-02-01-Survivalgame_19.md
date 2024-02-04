---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[18]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 창 컨트롤, 기본무기, 시간 컨트롤, 랜덤아이템 설정


## 창 컨트롤

- 저번 포스팅에 만들었던 능력치 업그레이드 창은 플레이어가 레벨업 했을 때만 등장하고, 업그레이드를 선택하고 나면 다시 사라지도록 해야합니다.

- LevelUp 오브젝트를 활성/비활성화 하여 만들수도 있겠지만 여기서는 Scale를 변화시키는 방법으로 구현했습니다.

### LevelUp.cs

- 창 컨트롤을 위한 스크립트를 준비합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LevelUp : MonoBehaviour
{
    RectTransform rect;

    void Awake()
    {
        rect = GetComponent<RectTransform>();    
    }
    
    public void Show()
    {
        rect.localScale = Vector3.one;  // 1,1,1
    }

    public void Hide()
    {
        rect.localScale = Vector3.zero;
    }
}

```

- 단순히 Scale을 키워 화면상에 보이게하도록하는 Show() 함수와 숨기게하는 Hide() 함수만으로 이루어져있습니다.
- 이 함수들은 GameManager에서 호출하여 사용될것입니다.
- LevelUp 오브젝트에 스크립트를 붙여줍니다.

#### GameManager.cs

```c#
[Header("# Game Object")]
public LevelUp uiLevelUp;

public void GetExp()
    {
        exp++;

        if(exp == nextExp[level])
        {
            level++;
            exp = 0;
            uiLevelUp.Show();
        }
    }
```

- 변수를 선언하고, 레벨업 했을 때 UI가 보이도록 Show()함수를 호출 했습니다.
- 반대로 Hide() 함수는 스크립트에서 부르는것이 아니라, 유니티 Item 0~4 오브젝트에서 호출합니다.
- 먼저 GameManager 오브젝트 변수에 LevelUp을 붙여준 뒤, Item 0~4 오브젝트를 선택하고 인스펙터창의 Button에서 OnClick 이벤트에 +버튼을 눌러 LevelUp 스크립트를 넣어줍니다. 그런다음 Hide() 함수를 선택해주면 됩니다.

![image](/images/2024-02-01/capture_1.png)


## 기본 무기 지급

- 게임을 실행 시 최초에 지급되는 기본무기를 설정합니다.
- 우선은 임시로 게임시작시 삽을 들고 시작하도록 했습니다.

### LevelUp.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LevelUp : MonoBehaviour
{
    RectTransform rect;
    Item[] items;

    void Awake()
    {
        rect = GetComponent<RectTransform>();
        items = GetComponentsInChildren<Item>(true);
    }
    
    public void Show()
    {
        rect.localScale = Vector3.one;  // 1,1,1
    }

    public void Hide()
    {
        rect.localScale = Vector3.zero;
    }
    
    // .. 게임시작시 버튼을 대신 눌러주는 함수
    public void Select(int index)
    {
        items[index].OnClick();
    }
}

```

- index를 받아 대신 OnClick() 이벤트를 실행하도록 했습니다.

### GameManager.cs

```c#
void Start()
    {
        health = maxHealth;

        // .. test
        uiLevelUp.Select(0);
    }
```

- 임시로 삽을 선택하도록 했습니다.
- 게임을 실행해보면 레벨업을 해서 능력치 업그레이드 UI가 나왔는데, 능력치 고를 시간도없이 계속해서 게임이 실행되고 있는 문제가 발생합니다. 

![image](/images/2024-02-01/capture_2.gif)

## 시간 컨트롤

- 시간과 관련된것은 GameManager에서 처리하면 좋습니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    // ... 매니저 인스턴스를 어디서든 접근 가능하게 함
    public static GameManager instance;

    [Header("# Game Object")]
    public bool isLive;
    public Player player;
    public PoolManager pool;
    public LevelUp uiLevelUp;

    [Header("# Game Control")]
    // ... 게임 시간과 최대게임시간 변수
    public float gameTime;
    public float maxGameTime = 2 * 10f;

    [Header("# Player Info")]
    // ... 체력, 레벨, 킬수, 경험치, 다음레벨에 필요한 경험치
    public int health;
    public int maxHealth = 100;
    public int level;
    public int kill;
    public int exp;
    public int[] nextExp = { 10, 30, 60, 100, 150, 210, 280, 360, 450, 600 };
    

    void Awake()
    {
        instance = this;
    }

     void Start()
    {
        health = maxHealth;

        // .. test
        uiLevelUp.Select(0);
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

        if(exp == nextExp[level])
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

- isLive 라는 변수를둬서 이 변수에 따라 게임을 멈추고 재개하는 함수를 만들었습니다.
- 단순히 Time의 timeScale을 변경시키는 방법으로 구현했습니다.
- 그리고 작성했던 스크립트들의 모든 Update 관련 함수에 isLive의 상태에따라 Update를 무시하도록 해줍니다.
- GameManager, Player, Enemy, Weapon, Spawner 스크립트의 Update 함수(FixedUpdate, LateUpdate포함)에 다음 로직을 추가합니다.


```c#
if (!GameManager.instance.isLive)
            return;
```



## 랜덤 아이템

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LevelUp : MonoBehaviour
{
    RectTransform rect;
    Item[] items;

    void Awake()
    {
        rect = GetComponent<RectTransform>();
        items = GetComponentsInChildren<Item>(true);
    }
    
    public void Show()
    {
        Next();
        rect.localScale = Vector3.one;  // 1,1,1
        GameManager.instance.Stop();
    }

    public void Hide()
    {
        rect.localScale = Vector3.zero;
        GameManager.instance.Resume();
    }
    
    public void Select(int index)
    {
        items[index].OnClick();
    }
    
    // .. 랜덤 활성화 함수
    void Next()
    {
        // 1. 모든 아이템 비활성화
        foreach(Item item in items)
        {
            item.gameObject.SetActive(false);
        }

        // 2. 그 중 랜덤 3개 아이템 활성화 
        int[] rand = new int[3];
        
        while(true)
        {
            rand[0] = Random.Range(0, items.Length);
            rand[1] = Random.Range(0, items.Length);
            rand[2] = Random.Range(0, items.Length);

            // .. 중복 아이템 제거
            if (rand[0] != rand[1] && rand[1] != rand[2] && rand[0] != rand[2])
                break;
        }

        for(int index=0;index<rand.Length;index++)
        {
            Item ranItem = items[rand[index]];

            // 3. 만렙 아이템은 소비아이템으로 대체
            if(ranItem.level == ranItem.data.damages.Length)
            {
                items[Random.Range(4,7)].gameObject.SetActive(true);    // .. 힐 아이템으로 대체
            }
            else
            {
                ranItem.gameObject.SetActive(true);
            }

        }

    }
}

```

- Next() 함수는 레벨업했을 때, UI에 보여질 아이템을 고르는 함수입니다.
- 지금은 5개인 아이템을 모두 비활성화 시킨 뒤, 3개를 골라 그 3개만 활성화 시키도록 했습니다.
- 그리고 만렙을 달성한 아이템은 나오면 안되므로 소비아이템으로 대체하는 로직이 들어있습니다. 
- 여기에서 소비아이템 4~6번을 나오도록 했는데, 지금은 힐 아이템밖에 없지만 나중에 추가될 수 있는 가능성을 고려해 넣었습니다.
- Next() 함수는 Show() 함수에서 호출하여 사용합니다.

### GameManager.cs

```c#
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
```

- 현재 제가 레벨을 총 10레벨까지로 만들어놓았는데, 최대 레벨이 된 후에도 무한히 레벨업을 할 수 있는 로직을 추가했습니다.
- Min()을 사용해 최대레벨이 되면 최대레벨의 경험치를 계속해서 사용하도록 했습니다.

#### HUD.cs

```c#
case InfoType.Exp:
                // .. 현재경험치 / 최대경험치
                float curExp = GameManager.instance.exp;
                float nextExp = GameManager.instance.nextExp[Mathf.Min(GameManager.instance.level, GameManager.instance.nextExp.Length - 1)];
                mySlider.value = curExp / nextExp;
                break;
```

- HUD 슬라이더에도 반영해줍니다.


![image](/images/2024-02-01/capture_3.gif)
