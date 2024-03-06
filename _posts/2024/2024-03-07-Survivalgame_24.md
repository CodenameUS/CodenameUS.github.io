---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[23]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 로직 보완

- 우선, 저번포스팅에서 BGM 로직을 작성하지않았는데 추후에 적당한 bgm을 찾으면 적용할것입니다.
- 기존 로직에 문제가있거나, 최적화가 필요한 부분을 보완합니다.

## Reposition.cs

- 기존에는 플레이어가 입력하는 키의 방향으로 재배치할 방향을 정하는 로직이었습니다.
- 이 방식은 플레이어의 입력방식에 따라 많이 차이가 나고 예외상황도 발생하여 변경합니다.

```c#
private void OnTriggerExit2D(Collider2D collision)
    {
        // ... 플레이어(Area)가 아니면 무시
        if (!collision.CompareTag("Area"))
            return;

        // ... 플레이어의 위치를 가져옴
        Vector3 playerPos = GameManager.instance.player.transform.position; 
        // ... 현재 타입맵의 위치를 가져옴
        Vector3 myPos = transform.position;


        // ... 타일맵/몬스터 위치 변경
        switch (transform.tag)
        {
            case "Ground":
                // .. 두 오브젝트의 위치 차이를 활용
                float diffX = playerPos.x - myPos.x;
                float diffY = playerPos.y - myPos.y;
                float dirX = diffX < 0 ? -1 : 1;
                float dirY = diffY < 0 ? -1 : 1;
                diffX = Mathf.Abs(diffX);
                diffY = Mathf.Abs(diffY);

                // ... 타일맵 위치를 플레이어가 가고 있는 방향 쪽으로 변경
                if (diffX > diffY)
                {
                    transform.Translate(Vector3.right * dirX * 40);     
                }
                else if (diffX < diffY)
                {
                    transform.Translate(Vector3.up * dirY * 40);
                }
                break;
            case "Enemy":
                if (coll.enabled)   // ... 몬스터가 살아있을 경우
                {
                    Vector3 dist = playerPos - myPos;
                    Vector3 ran = new Vector3(Random.Range(-3, 3), Random.Range(-3, 3), 0);
                    // ... 플레이어의 맞은편에서 나타나게하기
                    transform.Translate(ran + dist * 2);      
                }
                break;
        }
    }
```

- 이제는 두 오브젝트의 위치 차이를 활용하여 타일맵을 재배치하도록 했습니다.
- 몬스터도 마찬가지로, 플레이어와 몬스터의 위치차이를 활요하여 몬스터를 재배치하되, 랜덤한 방향에서 나타나도록 랜덤값을 주었습니다.


## 투사체 멈춤 보완

- 근접무기와 원거리무기 로직에 관통력(per)이 있습니다. 이 관통력은 근거리무기의경우 -1 로 지정하여 -1은 근접무기로 하기로 했습니다.
- 하지만 원거리무기의 per이 서서히 줄어들다 -1이 되면 투사체가 사라지지않고 멈추는 현상이 있어 이를 보완하려합니다.

### Weapon.cs

```c#
// ... Bullet 0 무기 배치 함수
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
            
            Vector3 rotVec = Vector3.forward * 360 * index / count;     // ... 무기 위치
            bullet.Rotate(rotVec);
            bullet.Translate(bullet.up * 1.5f, Space.World);    // ... 무기 배치                 
            bullet.GetComponent<Bullet>().Init(damage, -100, Vector3.zero);     // ... -100 값은 무한 관통(근접무기)
        }
    }
```

- 근접무기의 관통력(per)을 -100으로 변경했습니다.

### Bullet.cs

```c#
 // ... 변수 초기화
    public void Init(float damage, int per, Vector3 dir)
    {
        this.damage = damage;       // ... 왼쪽의 damge는 Bullet 클래스의 damage, 오른쪽은 매개변수의 damage
        this.per = per;

        // ... 관통이 무한보다 큰것(원거리무기)
        if(per >= 0)
        {
            rigid.velocity = dir * 15f;
        }
    }

    void OnTriggerEnter2D(Collider2D collision)
    {   
        if (!collision.CompareTag("Enemy") || per == -100)
            return;

        per--;

        // ... 불릿이 힘을 다했을 때
        if(per < 0)
        {
            rigid.velocity = Vector2.zero;
            gameObject.SetActive(false);
        }
    }
```

- per의 변경에 따라 Bullet 스크립트 로직도 조금 변경했습니다.

## 투사체 삭제

- 원거리 무기의 경우 적을향해 투사체를 날립니다. 그런데 만약 날아간 투사체가 몬스터를 충분히 맞히지 못하고 계속 날아가게 된다면
쓸데없는 투사체 오브젝트가 남아있게됩니다. 
- 따라서 투사체가 어느정도 날아가게 된다면 없애버리도록하는 로직을 추가했습니다.

### Bullet.cs

```c#
// .. 투사체 삭제
    private void OnTriggerExit2D(Collider2D collision)
    {
        // .. Player의 Area를 벗어나면 삭제
        if(!collision.CompareTag("Area") || per == -100)
            return;

        gameObject.SetActive(false);
    }
```

- OnTriggerExit2D 를 사용해서 투사체가 플레이어가 가지고있는 Area를 벗어나면 충분히 날아간것으로 생각하여 투사체를 비활성화 시키도록 했습니다.


## 레벨 디자인

- 이제 기존에 설계했던 필요 경험치로 되돌리고, 웨이브 레벨을 설정하는 로직이 하드코딩적이므로 바꾸어줍니다.

### Spawner.cs

```c#

public float levelTime;

void Awake()
    {
        spawnPoint = GetComponentsInChildren<Transform>();
        levelTime = GameManager.instance.maxGameTime / spawnData.Length;
    }
    
void Update()
    {
        if (!GameManager.instance.isLive)
            return;

        timer += Time.deltaTime;
        level = Mathf.Min(Mathf.FloorToInt(GameManager.instance.gameTime / levelTime), spawnData.Length - 1);  // ... Float to Integer

        // ... 레벨에 따른 몬스터 스폰
        if (timer > spawnData[level].spawnTime)
        {
            timer = 0f;
            Spawn();
        }
    }
```

- levelTime 변수를 두어, 웨이브를 levelTime에 의해 설정되도록 했습니다.