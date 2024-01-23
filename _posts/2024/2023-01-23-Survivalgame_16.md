---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[15]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 무기 및 장비 업그레이드

## 무기 업그레이드
- 플레이어의 자식 오브젝트로 있던 Weapon0, Weapon1을 지웁니다(게임 시작할 때는 무기가 없도록)


### Item.cs
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Item : MonoBehaviour
{
    public ItemData data;
    public int level;
    public Weapon weapon;

    Image icon;
    Text textLevel;

    void Awake()
    {
        icon = GetComponentsInChildren<Image>()[1];
        icon.sprite = data.itemIcon;

        Text[] texts = GetComponentsInChildren<Text>();
        textLevel = texts[0];
    }

    void LateUpdate()
    {
        textLevel.text = "Lv." + (level + 1);   
    }

    // .. 버튼별 클릭 이벤트
    public void OnClick()
    {
        switch (data.itemType)
        {
            case ItemData.ItemType.Melee:
            case ItemData.ItemType.Range:
                // .. level 0일 때
                if (level == 0)
                {
                    GameObject newWeapon = new GameObject();
                    weapon = newWeapon.AddComponent<Weapon>();
                    weapon.Init(data);
                }
                // .. 레벨 업
                else
                {
                    float nextDamage = data.baseDamage;
                    int nextCount = 0;

                    nextDamage += data.baseCount * data.damages[level];     
                    nextCount += data.counts[level];

                    weapon.LevelUp(nextDamage, nextCount);
                }
                break;
            case ItemData.ItemType.Glove:
                break;
            case ItemData.ItemType.Shoe:
                break;
            case ItemData.ItemType.Heal:
                break;
        }

        level++;

        // .. 최대레벨 도달 시 버튼 비활성화
        if(level == data.damages.Length)
        {
            GetComponent<Button>().interactable = false;
        }
    }
}

```

- 주요 변경점은 다음과 같습니다.
- OnClick 이벤트에 level이 0일 때(게임시작 시) 무기 레벨업 버튼을 누르면 무기가 생성되고(Player의 자식오브젝트로)
- level 1이상부터는 만들어두었던 Scriptable item data에 기반하여 데미지와 갯수(관통)을 높여가도록 했습니다.

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

    float timer;                // ... 공격 주기를 위한 타이머
    Player player;              // ... 플레이어 스크립트

    void Awake()
    {
        player = GameManager.instance.player;
    }

   
    void Update()
    {
        switch (id)
        {
            case 0:
                // ... Bullet 0 회전 로직
                transform.Rotate(Vector3.back * speed * Time.deltaTime);
                break;
            default:
                timer += Time.deltaTime;

                if (timer > speed)
                {
                    timer = 0f;
                    Fire();
                }
                break;
        }
        
        // ... 레벨업 테스트
        if (Input.GetButtonDown("Jump"))
        {
            LevelUp(10, 1);
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

    public void Init(ItemData data)
    {
        // .. basic set
        name = "Weapon " + data.itemId;
        transform.parent = player.transform;
        transform.localPosition = Vector3.zero;

        // .. property set
        id = data.itemId;
        damage = data.baseDamage;
        count = data.baseCount;

        for(int index=0;index < GameManager.instance.pool.prefabs.Length; index++)
        {
            if(data.projectile == GameManager.instance.pool.prefabs[index])
            {
                prefabId = index;
                break;
            }
        }
        switch (id)
        {
            case 0:
                speed = 150;
                Batch();
                break;
            default:
                speed = 0.3f;
                break;
        }
    }

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
            bullet.GetComponent<Bullet>().Init(damage, -1, Vector3.zero);     // ... -1 값은 무한 관통
        }
    }

    // ... Bullet 1 
    void Fire()
    {
        // ... 대상이 없다면
        if (!player.scanner.nearsetTarget)
            return;

        // ... 총알이 나가고자 하는 방향 설정
        Vector3 targetPos = player.scanner.nearsetTarget.position;
        Vector3 dir = targetPos - transform.position;       // ... 크기가 포함된 방향
        dir = dir.normalized;

        Transform bullet = GameManager.instance.pool.Get(prefabId).transform;
        // ... 위치와 회전 결정
        bullet.position = transform.position;
        // ... 지정된 축을 중심으로 목표를 향해 회전
        bullet.rotation = Quaternion.FromToRotation(Vector3.up, dir);
        // ... count는 관통력
        bullet.GetComponent<Bullet>().Init(damage, count , dir);
    }
}

```

- 주요변경점은 다음과같습니다. 
- Init() 함수의 매개변수로 Scriptable item data를 가져와서 초기화 할 수 있게끔 했습니다.
- 그리고 더이상 Start() 함수에서 Init() 함수를 실행할필요가 없기때문에 삭제했습니다.
- Awake() 함수에서 player 정보를 가져올 때, Weapon은 게임시작시 더이상 Player의 자식 오브젝트가 아니므로 GameManager에서 player를 찾도록 했습니다.

## 장비 업그레이드
- 장비(Gear)는 Glove와 Shoe가 있었습니다. Glove를 업그레이드하면 연사속도가증가하고, Shoe를 업그레이드하면 이동속도가 증가하도록 했습니다.

### Gear.cs

- Gear를 위한 스크립트를 작성합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Gear : MonoBehaviour
{
    public ItemData.ItemType type;
    public float rate;

    public void Init(ItemData data)
    {
        // .. basie set
        name = "Gear " + data.itemId;
        transform.parent = GameManager.instance.player.transform;
        transform.localPosition = Vector3.zero;

        // .. property set
        type = data.itemType;
        rate = data.damages[0];
        ApplyGear();
    }

    public void LevelUp(float rate)
    {
        this.rate = rate;
        ApplyGear();
    }

    
    void ApplyGear()
    {
        switch(type)
        {
            case ItemData.ItemType.Glove:
                RateUp();
                break;
            case ItemData.ItemType.Shoe:
                SpeedUp();
                break;
        }
    }

    // .. 연사속도 증가
    void RateUp()
    {
        Weapon[] weapons = transform.parent.GetComponents<Weapon>();

        foreach(Weapon weapon in weapons)
        {
            switch(weapon.id)
            {
                case 0:
                    weapon.speed = 150 + (150 * rate);
                    break;
                default:
                    weapon.speed = 0.5f * (1f - rate);
                    break;
            }
        }
    }

    // .. 이동속도 증가
    void SpeedUp()
    {
        float speed = 3;
        GameManager.instance.player.speed = speed + (speed * rate);
    }
}

```

- RateUp()함수는 연사속도를, SpeedUp()함수는 이동속도를 증가시킵니다.
- ApplyGear()함수를 호출하면 RateUp과 SpeedUp을 호출하여 적용시킵니다.

### Item.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Item : MonoBehaviour
{
    public ItemData data;
    public int level;
    public Weapon weapon;
    public Gear gear;

    Image icon;
    Text textLevel;

    void Awake()
    {
        icon = GetComponentsInChildren<Image>()[1];
        icon.sprite = data.itemIcon;

        Text[] texts = GetComponentsInChildren<Text>();
        textLevel = texts[0];
    }

    void LateUpdate()
    {
        textLevel.text = "Lv." + (level + 1);   
    }

    // .. 버튼별 클릭 이벤트
    public void OnClick()
    {
        switch (data.itemType)
        {
            case ItemData.ItemType.Melee:
            case ItemData.ItemType.Range:
                // .. level 0일 때
                if (level == 0)
                {
                    GameObject newWeapon = new GameObject();
                    weapon = newWeapon.AddComponent<Weapon>();
                    weapon.Init(data);
                }
                // .. 레벨 업
                else
                {
                    float nextDamage = data.baseDamage;
                    int nextCount = 0;

                    nextDamage += data.baseCount * data.damages[level];     
                    nextCount += data.counts[level];

                    weapon.LevelUp(nextDamage, nextCount);
                }

                level++;
                break;
            case ItemData.ItemType.Glove:
            case ItemData.ItemType.Shoe:
                if(level == 0)
                {
                    GameObject newGear = new GameObject();
                    gear = newGear.AddComponent<Gear>();
                    weapon.Init(data);
                }
                else
                {
                    float nextRate = data.damages[level];
                    gear.LevelUp(nextRate);
                }

                level++;
                break;
            case ItemData.ItemType.Heal:
                GameManager.instance.health = GameManager.instance.maxHealth;
                break;
        }

        // .. 최대레벨 도달 시 버튼 비활성화
        if(level == data.damages.Length)
        {
            GetComponent<Button>().interactable = false;
        }
    }
}

```

- Onclick 이벤트에서 Glove와 Shoe 부분을 추가했습니다. 그리고 Heal 아이템은 최대체력으로 체력을 회복시켜주도록 했습니다.
- Heal 아이템은 level이 없으므로 level++ 로직은 case문 안쪽으로 넣어줬습니다.

### Weapon.cs
- 마지막으로 Gear 장비를 업그레이드 했을 때, player가 가진 무기들에 ApplyGear()를 했는데, 현재 가지고 있지 않던 무기들에 대해서도
- 나중에 획득했을 때, Gear 업그레이드가 적용이 되는 로직을 추가했습니다.

```c#
        player.BroadcastMessage("ApplyGear",SendMessageOptions.DontRequireReceiver);  
```

- 위 코드를 LevelUp(), Init() 함수의 끝에 넣어주면 됩니다.
- 따라서 ApplyGear()는 4가지 경우에서 호출이됩니다. 첫번째로 weapon이 처음 생성될 때, 업그레이될 때, Gear가 처음 생성될 때, Gear가 업그레이드 되었을 때 호출됩니다.

![image](/images/2024/2024-01-23/capture_1.gif)
