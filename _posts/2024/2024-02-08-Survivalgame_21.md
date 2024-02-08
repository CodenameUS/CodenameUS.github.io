---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[20]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 캐릭터 선택

- 다양한 캐릭터를 선택하여 게임을 플레이할 수 있도록 변경합니다.


## 캐릭터 선택 UI

- 캐릭터 선택 UI는 게임시작시 보여야할 UI이므로, Canvas - GameStart 에 Create Empty를 추가하여 이름은 Character Group 이라고 지어줍니다.
- Grid Layout Group 컴포넌트를 추가해서 캐릭터선택 UI가 Grid 형식으로 보이도록 해주었습니다.

![image](/images/2024/2024-02-08/capture_1.png)


- 이전에 만든 Button Start를 Character Group의 자식 오브젝트로 넣어주고 이름을 Character 0으로 바꿔줍니다. 이제 게임시작 버튼은 캐릭터 선택버튼으로 바뀌었습니다.
- 캐릭터 선택버튼이므로 Image도 하나 추가해줍니다.
- Text (Lagacy)로 되어있던 오브젝트는 Text Name으로 바꾸고, 캐릭터 이름을 넣어줍니다.
- 이것을 복사하여 Text Adv로 이름을 지어주고, 캐릭터 속성을 넣어줍니다.

![image](/images/2024/2024-02-08/capture_2.png)

- Character 0 전체를 복사해서 Character 1, Character 2, Character 3까지 만들어줍니다.

 ![image](/images/2024/2024-02-08/capture_3.png)


## 캐릭터 선택 로직

- 캐릭터 선택창에서 캐릭터를 선택했을 때, 해당 캐릭터로 게임을 플레이할 수 있도록합니다.
- 먼저 GameManager 스크립트에서 PlayerInfo에 playerId를 추가해줍니다.

### GameManager.cs

```c#
 [Header("# Player Info")]
 public int playerId;

 public void GameStart(int id)
    {
        playerId = id;
        health = maxHealth;

        player.gameObject.SetActive(true);
        uiLevelUp.Select(playerId % 2);   // .. 기본 무기 지급
        Resume();       
    }
```

- GameStart 함수에서 게임시작 시 id를 받아 playerId를 지정하고, playerId에 맞는 플레이어 오브젝트를 활성화 시켜주도록 했습니다.
- 기존에 기본무기(삽)을 지급하던것을 playerId에 따라 지급하도록 했습니다.(현재는 무기가 2종류밖에 없으므로 %2를 해서 무기 지급이 안되는것을 방지했습니다)
- 이제 Player는 비활성화 시켜줍니다.

### Player.cs

- 이제 Player에서도 playerId에 맞는 애니메이션컨트롤러를 가지도록 해주어야합니다.
- 예전에 만들어뒀던 프로젝트폴더의 Animation 폴더에서 Ac Player 0~3를 사용할 때가 왔습니다.

```c#
 public RuntimeAnimatorController[] animCon;

 void OnEnable()
    {
        anim.runtimeAnimatorController = animCon[GameManager.instance.playerId];
    }
```

- 애니메이션 컨트롤러를 담을 변수를 추가해주고, 플레이어가 활성화됐을 때 해당 컨트롤러로 바꾸어주도록 했습니다.

 ![image](/images/2024/2024-02-08/capture_4.png)

- 마지막으로 Canvas - Character0 ~ 3까지 Button의 Onclick() 이벤트의 GamaManager - GameStart함수를 선택하고 해당 캐릭터에 맞는 playerId를 부여합니다.

 ![image](/images/2024/2024-02-08/capture_5.png)


## 캐릭터 특성 구현

- 캐릭터 특성은 따로 스크립트를 만들어 속성을 작성해보겠습니다.

### Character.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Character : MonoBehaviour
{
    // .. 이동속도
    public static float Speed
    {
        get { return GameManager.instance.playerId == 0 ? 1.1f : 1f; }
    }

    // .. 발사속도
    public static float WeaponSpeed
    {
        get { return GameManager.instance.playerId == 1 ? 1.1f : 1f; }
    }

    // .. 연사속도
    public static float WeaponRate
    {
        get { return GameManager.instance.playerId == 1 ? 0.9f : 1f; }
    }

    // .. 데미지
    public static float Damage
    {
        get { return GameManager.instance.playerId == 2 ? 1.2f : 1f; }
    }
    
    // .. 발사체, 관통력추가
    public static int Count
    {
        get { return GameManager.instance.playerId == 3 ? 1 : 0; }
    }
}

```

- Player의 Id에 따라 캐릭터 특성들을 반영하도록 했습니다.

### Player.cs

```c#
void OnEnable()
    {
        speed *= Character.Speed;
        anim.runtimeAnimatorController = animCon[GameManager.instance.playerId];
    }
```

- Player가 활성화 됐을 때 speed를 특성에 맞게 설정해줍니다.

### Weapon.cs

```c#
 // ... 레벨업 함수
    public void LevelUp(float damage, int count)
    {
        this.damage = damage * Character.Damage;
        this.count += count;

        if (id == 0)        // .. 근접무기
            Batch();

        // player가 가진 모든 gear에 대해 ApplyGear 실행
        player.BroadcastMessage("ApplyGear", SendMessageOptions.DontRequireReceiver);
    }

    public void Init(ItemData data)
    {
        // .. basic set
        name = "Weapon " + data.itemId;
        transform.parent = player.transform;
        transform.localPosition = Vector3.zero;

        // .. property set
        id = data.itemId;
        damage = data.baseDamage * Character.Damage;
        count = data.baseCount + Character.Count;

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
                speed = 150 * Character.WeaponSpeed;
                Batch();
                break;
            default:
                speed = 0.3f * Character.WeaponRate;
                break;
        }

        // .. Hand Set
        Hand hand = player.hands[(int)data.itemType];   // .. 정수형 캐스팅
        hand.spriter.sprite = data.hand;
        hand.gameObject.SetActive(true);

        player.BroadcastMessage("ApplyGear",SendMessageOptions.DontRequireReceiver);  
    }
```

- Init() 함수와 LevelUp 함수에서 특성들을 적용해주었습니다.

### Gear.cs

```c#
// .. 연사속도 증가
    void RateUp()
    {
        Weapon[] weapons = transform.parent.GetComponents<Weapon>();

        foreach(Weapon weapon in weapons)
        {
            switch(weapon.id)
            {
                case 0:
                    float speed = 150 * Character.WeaponSpeed;
                    weapon.speed = speed + (speed * rate);
                    break;
                default:
                    speed = 0.5f * Character.WeaponRate;
                    weapon.speed = 0.5f * (1f - rate);
                    break;
            }
        }
    }

    // .. 이동속도 증가
    void SpeedUp()
    {
        float speed = 3 * Character.Speed;
        GameManager.instance.player.speed = speed + (speed * rate);
    }
```

- Gear 스크립트에서도 특성을 반영했습니다.