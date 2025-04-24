---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[7]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 오브젝트 풀링 개념

- 게임을 플레이할 때 몬스터들이 사방에서 무한정으로 생성되는것을 만들기 위하여 몬스터라는 오브젝트를 계속 생성하고 없애주는 방식을 사용하게 되면, 메모리가 부족해지는 문제가 발생할 수 있습니다.

- 따라서 유니티에서는 풀링(Pooling) 이라는 기법을 사용할 수 있습니다.

- 오브젝트 풀링이란, 쉽게 말해 풀(Pool)을 만들어 그 안에 오브젝트들을 넣어 보관하고, 필요할 때 가져다 쓰며 사용한 뒤에는 다시 풀에 돌려놓는 방식을 말합니다.

- 그러므로 무한정으로 몬스터들을 생성/삭제하는것이 아닌, 필요하면 추가적으로 생성/삭제하여 오브젝트의 생성/삭제 횟수를 줄일 수 있게 됩니다.


## 몬스터 프리팹 생성

- Undead Survival 폴더 아래에 Prefabs 이름의 폴더를 하나 생성합니다.

- 이 폴더로, Hierarchy 뷰에있는 몬스터 오브젝트를 끌어다 놓게되면 프리팹이 생성됩니다.

![image](/images/2023/2023-09-18/capture_1.png)


- 이제 Hierarchy 뷰에있는 몬스터 오브젝트들을 전부 삭제해주고, Create Empty - "PoolManager" 라는 이름의 오브젝트를 만들어줍니다.

- Scripts 폴더에도 마찬가지로 PoolManager 라는 이름의 스크립트를 생성합니다.

- 스크립트를 열어 프리팹을 관리할 수 있도록 코드를 작성합니다.


### PoolManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PoolManager : MonoBehaviour
{
    // .. 프리팹을 보관할 변수
    public GameObject[] prefabs;

    // .. 풀 담당할 리스트
    List<GameObject>[] pools;

    void Awake()
    {
        // .. 리스트 초기화
        pools = new List<GameObject>[prefabs.Length];
        
        // .. 풀 리스트 요소 초기화
        for (int index = 0; index < pools.Length; index++)
        {
            pools[index] = new List<GameObject>();
        }
    }

    public GameObject Get(int index)
    {
        GameObject select = null;

        // ... 선택한 풀에서 비활성화 된 게임오브젝트 접근
        foreach(GameObject item in pools[index])
        {
            if (!item.activeSelf)
            {
                // ... 있을 때 select 변수에 할당
                select = item;
                select.SetActive(true);
                break;
            }
        }
        // ... 못 찾았을 때
        if (select == null)
        {
            // ... 새롭게 생성 하고 select 변수에 할당
            // ... transform 인자는 Hierarchy 뷰에 지저분하게 보이지 않도록 PoolManager 아래에 생성하기위함
            select = Instantiate(prefabs[index], transform);

            // ... 생성된 오브젝트를 Pool에 추가
            pools[index].Add(select);
        }
        return select;
    }
}
```

1. 앞에서 설명했듯이 오브젝트를 관리할 풀(리스트)과, 오브젝트가 필요하므로 변수를 만들어줍니다.

2. 여러개의 프리팹들을 사용할것이기 때문에 배열의 형태로 선언하였습니다.

3. 풀(리스트)를 초기화 해주고, 리스트 안의 요소들도 함께 초기화 합니다.

4. 풀링을 하기위한 함수로, Get 이라는 함수를 만들어 주었습니다. 풀(리스트)안에 사용가능한 오브젝트가 있을 때와 없을 때의 경우를 나누어 생각합니다.

5. 있을때는 select라는 변수에 담아 반환하고, 없을때는 새롭게 만든 뒤 풀안에 추가하고 반환합니다.

6. 반환되는 형식은 게임오브젝트입니다.


- 이 스크립트를 PoolManager에 붙여넣고, Prefabs 폴더에 있는 프리팹들을 연결시켜줍니다.

![image](/images/2023/2023-09-18/capture_2.png)


### Spawner.cs

- 풀링을 하기위한 작업은 완료가 되었으니 실제로 오브젝트를 생성해보겠습니다.

- 테스트를 위해 Player의 자식 오브젝트로 Spawner라는 이름의 오브젝트를 만들어줍니다.

- 마찬가지로 Scripts 폴더에 Spawner 라는 이름의 스크립트를 생성한 뒤 Spawner 오브젝트에 붙여줍니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Spawner : MonoBehaviour
{

    void Update()
    {
        if (Input.GetButtonDown("Jump"))
        {
            GameManager.instance.pool.Get(0);
        }
    }
}

```

1. 먼저, GameObject.cs 에서 PoolManager를 다른곳에서 접근할 수 있도록 추가합니다.

![image](/images/2023/2023-09-18/capture_3.png)


2. 플레이어가 Jump키(Space바)를 누르면 PoolManager의 Get함수를 불러와 몬스터를 Scene에 생성하게됩니다.

![image](/images/2023/2023-09-18/capture_4.gif)

- 하지만 이렇게만 하면 Jump키를 누를때마다 오류가 발생합니다.

- 그 이유는 Enemy.cs에 있는데, 우리가 Scene에 올라와있는 오브젝트와 Prefabs 폴더에 있는 애셋들은 따로 분리되어 있어서 현재 Enemy.cs에 작성되어 있는 방법으로는 target.. 즉 플레이어를 생성과 동시에 찾을 수 없어서 발생하는 것입니다.

- 따라서 Enemy.cs를 수정해주어야 합니다.

### Enemy.cs

```c#
// ... 프리팹이 생성 될 때 Target(플레이어) 찾기
    private void OnEnable()
    {
        target = GameManager.instance.player.GetComponent<Rigidbody2D>();
    }
```

- 새로운 함수를 정의하여 몬스터가 생성될 때 target을 찾을 수 있도록 수정했습니다.

- OnEnable() 함수는 오브젝트가 활성화 될 때 한번 실행되는 함수입니다.

- 수정하고나면 더이상 오류는 발생하지 않습니다.

- 이제 자동으로 몬스터가 생성되도록 해볼 차례입니다.


## Spawner.cs 수정

- 좀 더 게임같게 만들기 위하여 몬스터가 플레이어 주위에 자동으로 생성되도록 해보겠습니다.

- 플레이어 아래의 Spawner 오브젝트에 Create Empty 하여 Point 라는 이름의 오브젝트를 만들어줍니다.

- 이 오브젝트는 몬스터가 소환될 위치입니다.

- 카메라 바깥쪽에서 몬스터가 생성되어 플레이어쪽으로 다가오는게 자연스러울테니, 아래와같이 스폰 위치를 잡아줍니다.


![image](/images/2023/2023-09-18/capture_5.png)


```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Spawner : MonoBehaviour
{
    public Transform[] spawnPoint;

    float timer;

    private void Awake()
    {
        spawnPoint = GetComponentsInChildren<Transform>();
    }
    void Update()
    {
        timer += Time.deltaTime;

        if (timer > 0.2f)
        {
            Spawn();
            timer = 0f;
        }
    }

    void Spawn()
    {
        GameObject enemy = GameManager.instance.pool.Get(Random.Range(0,2));
        enemy.transform.position = spawnPoint[Random.Range(1, spawnPoint.Length)].position;
    }
}

```

- 스폰 위치를 변수 spawnPoint, 소환 주기를 맞출 timer 변수를 만들어주었습니다.

- 스폰 위치를 초기화 해주는데, 조금 다른것이 GetComponentsInChildren입니다. 부모(플레이어)의 자식 오브젝트들로써 초기화되는것입니다.

- timer 변수는 프레임당 시간을 더해주고, 0.2초마다 한번씩 몬스터가 스폰되어 나오게되도록 했습니다. 따라서 1초에 5마리씩 나오게됩니다.

- Spawn() 함수에서는 스폰될 몬스터의 종류(현재는 2가지를 추가하였음)를 랜덤으로설정, 몬스터의 위치도 랜덤(아까 설정한 Point들 중 하나)하게 나오도록 설정했습니다.

![image](/images/2023/2023-09-18/capture_6.gif)
