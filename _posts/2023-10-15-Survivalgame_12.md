---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[11]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 몬스터 애니메이션 추가

- 몬스터에 대한 애니메이션은 골드메탈님께서 이미 Animation-Enemy 폴더에 만들어두셨습니다.
- 플레이어의 공격에 맞으면 뒤로 살짝 밀려나고, 죽었을때는 죽음 애니메이션을 실행하도록 합니다.

## Enemy.cs

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
    WaitForFixedUpdate wait;
    Collider2D coll;

    void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
        sprite = GetComponent<SpriteRenderer>();
        anim = GetComponent<Animator>();
        coll = GetComponent<Collider2D>();
        wait = new WaitForFixedUpdate();
    }

    // ... 프리팹이 생성 될 때 Target(플레이어) 찾기
    private void OnEnable()
    {
        target = GameManager.instance.player.GetComponent<Rigidbody2D>();
        isLive = true;
        coll.enabled = true;
        rigid.simulated = true;    // ... 리지드바드 비활성화는 simulated
        sprite.sortingOrder = 2;
        anim.SetBool("Dead", false);

        health = maxHealth;     // ... hp 원상복구
    }

    void FixedUpdate()
    {
        // ... Hit 상태이면 무시
        if (!isLive || anim.GetCurrentAnimatorStateInfo(0).IsName("Hit"))
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

    // ... 불릿과 충돌 이벤트
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (!collision.CompareTag("Bullet") || !isLive)
            return;

        health -= collision.GetComponent<Bullet>().damage;
        StartCoroutine(KnockBack());

        if(health > 0)
        {
            // ... 체력이 남아있을 경우
            anim.SetTrigger("Hit"); // ... 애니메이션
        }
        else  // ... 죽었을 때
        {
            isLive = false;
            coll.enabled = false;
            rigid.simulated = false;    // ... 리지드바드 비활성화는 simulated
            sprite.sortingOrder = 1;
            anim.SetBool("Dead", true);

            // ... 킬수증가, exp증가
            GameManager.instance.kill++;
            GameManager.instance.GetExp();
        }
    }

    // ... 코루틴 : 생명주기와 비동기처럼 실행되는 함수
    IEnumerator KnockBack()
    {
        yield return wait; // ... 다음 하나의 물리 프레임 딜레이
        Vector3 playerPos = GameManager.instance.player.transform.position; // ... 플레이어 위치
        Vector3 dirVec = transform.position - playerPos;    // ... 플레이어 기준 반대방향
        rigid.AddForce(dirVec.normalized * 3, ForceMode2D.Impulse); // ... 반대방향으로 힘 가하기
    }

    void Dead()
    {
        gameObject.SetActive(false);
    }
}

```

- 코루틴을 사용해서, 넉백효과를 만들어봤습니다.
- 몬스터가 죽었을 때는 다른 몬스터들과 충돌하지않도록 해주고, layer order를 하나 내려줌으로써 살아있는 몬스터들을 가리지 않도록 했습니다.
- 몬스터가 다시 생성되었을 때는 반대로 레이어,충돌 등을 원래대로 돌려놓아야하므로 OnEnable 함수에서 돌려놓았습니다.
- GameManager에서 킬수와 경험치를 다루는 내용도 추가되었습니다. 아래에서 다루도록합니다.


## GameManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    // ... 매니저 인스턴스를 어디서든 접근 가능하게 함
    public static GameManager instance;

    [Header("# Game Object")]
    public Player player;
    public PoolManager pool;

    [Header("# Game Control")]
    // ... 게임 시간과 최대게임시간 변수
    public float gameTime;
    public float maxGameTime = 2 * 10f;

    [Header("# Player Info")]
    // ... 레벨, 킬수, 경험치, 다음레벨에 필요한 경험치
    public int level;
    public int kill;
    public int exp;
    public int[] nextExp = { 10, 30, 60, 100, 150, 210, 280, 360, 450, 600 };
    

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

    public void GetExp()
    {
        exp++;

        if(exp == nextExp[level])
        {
            level++;
            exp = 0;
        }
    }
}

```

- 카테고리(헤더)를 달아서 인스펙터창에서 좀 더 가시성있도록 만들었습니다.
- nextExp는 다음 레벨에 도달하기 위한 경험치로, 임의로 설정하였습니다.
- Enemy 스크립트에서 몬스터가 죽을 때마다 킬수와 경험치가 오르도록 했습니다.

### 애니메이터 설정

- 스크립트를 작성하고, 몬스터의 Dead애니메이션을 더블클릭합니다.
- 다음과 같이 1초의 텀을 두고 애니메이션이 동작하도록 설정합니다.

![image](/images/2023-10-15/capture_1.png)

- 그런다음, 왼쪽 상단의 Add event 버튼을 눌러 애니메이션 이벤트를 추가합니다.
- 이제 애니메이션창을 띄워둔 채로 몬스터 프리팹을 더블클릭하고 Hierarchy 뷰의 Enemy를 선택하면 이벤트 함수를 선택할 수 있는 창이 인스펙터창에 보이게됩니다. 여기에서 Dead() 함수를 선택해줍니다.

![image](/images/2023-10-15/capture_2.png)
