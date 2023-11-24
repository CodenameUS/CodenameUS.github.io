---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[8]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 소환 레벨 적용하기

- 소환레벨이란 게임 진행시간에따라서 더 많은 적이 소환되도록 설정하는 레벨이며, 몬스터 웨이브라고 생각하면 됩니다.

- GameManager, Spawner, Enemy 세 개의 스크립트를 수정 보완하여 구현해보겠습니다.


## GameManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public Player player;
    public PoolManager pool;

    // ... 게임 시간과 최대게임시간 변수
    public float gameTime;
    public float maxGameTime = 2 * 10f;

    // ... 매니저 인스턴스를 어디서든 접근 가능하게 함
    public static GameManager instance;
    

    void Awake()
    {
        instance = this;
    }

    
    void Update()
    {
        gameTime += Time.deltaTime;

        if (gameTime > maxGameTime)
        {
            gameTime = maxGameTime;
        }
    }
}

```

- 두 개의 변수(gameTime, maxGameTime)를 추가했습니다.

## Spawner.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Spawner : MonoBehaviour
{
    // ... 스폰위치를 담을 변수
    public Transform[] spawnPoint;
    // ... 스폰데이터를 담을 변수
    public SpawnData[] spawnData;

    // ... 소환 주기를 위한 타이머
    float timer;

    // ... 소환 웨이브 레벨 변수
    int level;


    private void Awake()
    {
        spawnPoint = GetComponentsInChildren<Transform>();
    }
    void Update()
    {
        timer += Time.deltaTime;
        level = Mathf.FloorToInt(GameManager.instance.gameTime / 10f);  // ... Float to Integer

        // ... 레벨에 따른 몬스터 스폰
        if (timer > spawnData[level].spawnTime)
        {
            timer = 0f;
            Spawn();
        }
    }

    // ... 몬스터 스폰 함수
    void Spawn()
    {
        // ... 레벨에 따라 몬스터 소환
        GameObject enemy = GameManager.instance.pool.Get(0);
        // ... 몬스터 위치를 지정한 위치중 랜덤으로 정함
        enemy.transform.position = spawnPoint[Random.Range(1, spawnPoint.Length)].position;
        enemy.GetComponent<Enemy>().Init(spawnData[level]);
    }
}

// ... 소환 데이터 담당 클래스
[System.Serializable]   // ... 직렬화  => 인스펙터창에서 보임
public class SpawnData
{
    public int spriteType;
    public int health;
    public float spawnTime;
    public float speed;
}
```

- SpawnData라는 새로운 클래스를 정의하여 이 안에 스폰데이터를 집어넣도록 했습니다.
- [System.Serializable] 는 인스펙터창에서 이 클래스에 데이터를 집어넣을 수 있도록 보여지게하는 직렬화 기능입니다.
- Update 함수에 보면 level 이라는 변수에 게임시간에 따른 레벨을 부여하여 일정 시간이상이 지나면 Spawn함수에서 level에 따른 
몬스터를 소환하도록 만들어줬습니다.

![image](/images/2023-10-02/capture_1.png)

- 인스펙터창의 Spawner를 보게되면 이런식으로 속성값을 따로 코딩없이 넣을 수 있게된것을 볼 수 있습니다.[데이터는 현재 임시로 집어넣음]


## 몬스터 다듬기 및 Enemy.cs

- 프리팹 폴더에 있는 몬스터 프리팹을 하나만 남기고 지워버리겠습니다.

![image](/images/2023-10-02/capture_2.png)

- Speed를 0으로 해줍니다.


```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Enemy : MonoBehaviour
{
    public float speed;         // ... 몬스터 속도
    public float health;        // ... 현재체력
    public float maxHealth;     // ... 최대체력

    public RuntimeAnimatorController[] animCont;    // ... 애니메이터 컨트롤러
    public Rigidbody2D target;  // ... 타겟(플레이어)

    bool isLive;              // ... 몬스터 생존여부

    Rigidbody2D rigid;
    Animator anim;
    SpriteRenderer sprite;

    void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
        sprite = GetComponent<SpriteRenderer>();
        anim = GetComponent<Animator>();
    }

    // ... 프리팹이 생성 될 때 Target(플레이어) 찾기
    private void OnEnable()
    {
        target = GameManager.instance.player.GetComponent<Rigidbody2D>();
        isLive = true;
        health = maxHealth;     // ... hp 원상복구
    }

    void FixedUpdate()
    {
        if (!isLive)
            return;
       
        // ... 플레이어와 몬스터 위치 차이
        Vector2 dirVec = target.position - rigid.position;  
        // ... 몬스터 다음 움직일 방향 설정
        Vector2 nextVec = dirVec.normalized * speed * Time.fixedDeltaTime;
        rigid.MovePosition(rigid.position + nextVec);
        rigid.velocity = Vector2.zero;
    }

    private void LateUpdate()
    {
        if (!isLive)
            return;
        // ... 몬스터가 바라보는 방향설정(애니메이션)
        sprite.flipX = target.position.x < rigid.position.x;    
    }

    // ... 몬스터 초기화
    public void Init(SpawnData data)
    {
        anim.runtimeAnimatorController = animCont[data.spriteType];
        speed = data.speed;
        maxHealth = data.health;
        health = data.health;
    }
}

```

- 몬스터의 애니메이터 컨트롤러를 추가합니다.

- 몬스터의 최대체력, 현재체력을 담을 변수도 함께 추가합니다.

![image](/images/2023-10-02/capture_3.png)

- 몬스터 스크립트에 애니메이션 컨트롤러 부분이 생긴것을 볼 수 있습니다. 여기에서 몬스터 종류를 추가하면 됩니다.

- 몬스터 데이터를 초기화하기위한 부분은 Init함수에서 진행하고, Spawner 스크립트에서 이를 호출해 사용하게 됩니다.



![image](/images/2023-10-02/capture_4.gif)
