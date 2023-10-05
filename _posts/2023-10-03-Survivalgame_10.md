---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[9]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 무기 구현하기


## 불릿만들기

1. 프로젝트의 Sprites 폴더에있는 Props의 Bullet0 스프라이트를 캐릭터 주위에 배치합니다.

2. Add Tag에서 "Bullet" 이름으로 태그를 하나 만든 뒤 Bullet 오브젝트 태그를 설정합니다.

3. "Bullet" 이름으로 스크립트를 만들어 붙여줍니다.

![image](/images/2023-10-03/capture_1.png)


### Bullet.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bullet : MonoBehaviour
{

    public float damage;        // ... 총알 데미지
    public int per;             // ... 관통

    // ... 변수 초기화
    public void Init(float damage, int per)
    {
        this.damage = damage;       // ... 왼쪽의 damge는 Bullet 클래스의 damage, 오른쪽은 매개변수의 damage
        this.per = per;

    }
}

```

- 불릿의 데미지와 관통변수를 넣어줍니다. 이번 포스팅에서는 근접무기만 다루어볼것이므로 관통은 신경쓰지 않습니다.
- Init() 함수에서 변수를 초기화합니다.

## 프리팹 만들기

- 무기 프리팹을 만들어봅니다. 아까 배치해두었던 Bullet 오브젝트를 프리팹 파일로 끌어놓아 프리팹을 생성합니다.
- 몬스터와의 충돌이벤트를위해 이 프리팹에 BoxCollider2D 컴포넌트를 달아주고 IsTrigger를 체크합니다.

![image](/images/2023-10-03/capture_2.png)


### Enemy.cs

- 무기를 만들었으니 동작하는지 확인하기위해 Enemy 스크립트에서 충돌이벤트를 구현해줍니다.

```c#
// ... 불릿과 충돌 이벤트
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (!collision.CompareTag("Bullet"))
            return;

        health -= collision.GetComponent<Bullet>().damage;

        if(health > 0)
        {
            // ... 체력이 남아있을 경우
        }
        else
        {
            // ... 사망
            Dead();
        }
    }

    void Dead()
    {
        gameObject.SetActive(false);
    }
```

- Enemy.cs 의 아랫부분에 다음 코드를 작성합니다.

- OnTriggerEnter2D에서 Bullet과 충돌했는지 검사하고, 충돌했을때 hp를 깎습니다. 이때 Bullet 스크립트의 데미지를 가져와
몬스터의 체력에서 빼는식으로 만들었습니다.

- 두가지 경우가 있을텐데, 첫번째로 몬스터가 데미지를 받고나서도 살아있을 경우, 두번째로 죽을경우입니다.

- 살아있을 경우는 후에 다루도록하고, 죽었을 경우에는 Dead()함수를 호출하여 비활성화합니다.

## 무기 배치

- 무기를 만들고, 충돌로직까지 작성해봤습니다. 이제 무기를 캐릭터주위에 예쁘게 배치하는작업을 합니다.

- 플레이어 자식으로 "Weapon 0" 이름의 오브젝트를 생성합니다. "Weapon" 이름으로 스크립트도 하나 생성합니다.

![image](/images/2023-10-03/capture_3.png)


### Weapon.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Weapon : MonoBehaviour
{
    public int id;              // ... 무기 id
    public int prefabId;        // ... 프리팹 id
    public float damage;        // ... 무기데미지
    public int count;           // ... 개수
    public float speed;         // ... 속도

    void Start()
    {
        Init();
    }

    void Update()
    {
        switch (id)
        {
            case 0:
                // ... 무기 회전 로직
                transform.Rotate(Vector3.back * speed * Time.deltaTime);
                break;
            default:
                break;
        }
        
        // ... 레벨업 테스트
        if (Input.GetButtonDown("Jump"))
        {
            LevelUp(20, 5);
        }
    }

    // ... 레벨업 함수
    public void LevelUp(float damage, int count)
    {
        this.damage = damage;
        this.count += count;

        if (id == 0)        // .. 근접무기
            Batch();
    }

    public void Init()
    {
        switch (id)
        {
            case 0:
                speed = 150;
                Batch();
                break;
            default:
                break;
        }
    }

    // ... 무기 배치 함수
    void Batch()
    {
        for(int index=0; index < count; index++)
        {
            // ... 불릿의 위치정보
            Transform bullet;

            if (index < transform.childCount)
            {
                bullet = transform.GetChild(index);
            }
            else
            {
                bullet = GameManager.instance.pool.Get(prefabId).transform;
                // ... 부모를 자식으로
                bullet.parent = transform;
            }

            bullet.localPosition = Vector3.zero;
            bullet.localRotation = Quaternion.identity;
            
            Vector3 rotVec = Vector3.forward * 360 * index / count;     // ... 무기 방향
            bullet.Rotate(rotVec);                              // ... 무기 방향 설정
            bullet.Translate(bullet.up * 1.5f, Space.World);    // ... 무기 위치 설정                 
            bullet.GetComponent<Bullet>().Init(damage, -1);     // ... -1 값은 무한 관통
        }
    }
}

```

- 로직이 복잡해보입니다. 하나하나 따져보겠습니다.

- 5가지 변수가 있습니다. 무기번호, 프리팹번호, 데미지, 속도, 개수가 있습니다.

- Init() 함수에서는 무기 id에 따라 초기화를 합니다. 현재는 근접무기를 구현중이므로 무기의 스피드와 Batch() 함수를통한 무기의 위치 배치를 해줍니다.

- Update() 함수에서는 무기회전을 시켜줍니다. 프레임마다 스피드에따라 Rotate 시켜 회전합니다.

- Batch() 함수는 근접무기의 배치를 고려합니다. 무기 개수에따라 캐릭터 주위의 무기위치가 달라지게되는데, 이 위치는 360도를 기준으로 count(무기개수)로 나눈 위치에 생겨나도록 합니다.

- 레벨업을 할 경우를 대비하여 무기의 localPosition을 0으로 초기화하는 부분이있습니다. 이렇게 하지 않으면 레벨업했을 때 무기위치가 원치않는곳에 생겨나게됩니다.

- 마지막으로 LevelUp 함수에서는 damage와 count를 받아 그만큼 데미지와 개수를 늘려줍니다.

- 테스트를 위해 스페이스바를 누르면 레벨업이 되도록 설정했습니다.

## 테스트

1. Weapon 0 오브젝트에 Weapon 스크립트를 붙이고 값을 설정해주었습니다. (첫번째 웨이브 몬스터가 한번에 죽을정도로설정함)

![image](/images/2023-10-03/capture_4.png)

2. 풀링을 위해 풀매니저에 불릿을 등록합니다.

![image](/images/2023-10-03/capture_5.png)

3. 게임을 실행해서 작동되는지 확인합니다.

![image](/images/2023-10-03/capture_6.gif)
