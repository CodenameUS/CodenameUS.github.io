---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[17]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 레벨업 시스템

- 레벨업 했을 때, 랜덤한 능력치를 업그레이드 할 수 있는 UI창을 띄워 플레이어가 능동적으로 플레이할 수 있도록 합니다.


## UI 완성

- 먼저 레벨업 했을 때 보여질 능력치 업그레이드 창을 만들어보겠습니다.
- Canvas - UI - Image를 하나 만들고, 검은색 바탕에 Alpha 값을 낮추어 UI에 집중할 수 있도록 해주었습니다.

![image](/images/2024/2024-01-31/capture_1.png)

- 기존에 있던 LevelUp 오브젝트의 이름은 이제 ItemGroup이 되고, 방금만든 Image 오브젝트는 LevelUp으로 이름을 변경했습니다.
- LevelUp 자식오브젝트로 다시 Image를 하나 만들어 아래와 같은 느낌으로 만들었습니다.

![image](/images/2024/2024-01-31/capture_2.png)


- 이제 ItemGroup 오브젝트를 LevelUp 오브젝트의 자식 오브젝트로 넣어줍니다.
- Anchor 설정은 꽉찬 화면으로 지정합니다. 

![image](/images/2024/2024-01-31/capture_3.png)


- ItemGroup의 Vertical Layout을 보면 Control Child Size 버튼을 통해 자식 오브젝트들(Item0~4)을 자신의 크기에 맞게 자동으로 변경할 수 있는 기능이 있습니다.

- 각 버튼마다 약간의 여백을 주기위해 Spacing은 1정도로, Transform도 적절히 변경한 뒤, 레벨업 했을 때 올릴 수 있는 능력치는 한번에 3개를 보여줄 것이므로 Item3과 4는 잠깐 비활성화 해둡니다.


![image](/images/2024/2024-01-31/capture_4.png)


- 아이템에 대한 설명(텍스트)를 넣어주기 위해 Icon 위치를 살짝 움직여주고,  텍스트를 추가합니다.

![image](/images/2024/2024-01-31/capture_5.png)



### 아이템 텍스트넣기

- 프로젝트 폴더의 Data 폴더에 Scriptable 데이터를 눌러보면, Item Desc를 아직 비워둔 상태입니다.
- 설명란을 여러줄로 적으려면 속성하나를 부여하면됩니다.

#### ItemData.cs

```c#
 [TextArea]
    public string itemDesc;
```

- itemDesc 변수 위에 [Text Area]라는 속성을 주면, 여러줄에 텍스트를 적을 수 있게됩니다.
- 그 다음, ItemDesc란에 아이템 설명을 작성합니다. 설명에 데이터가 들어가는 자리에는 {index} 형태로 작성하여 데이터를 받아올 수 있게했습니다.
- 각 ItemData에 알맞은 설명을 적어줍니다.

![image](/images/2024/2024-01-31/capture_6.png)


- 이제 이 Text들을 게임상에서 보여주기 위해 Item 스크립트를 수정합니다.

#### Item.cs

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
    Text textName;
    Text textDesc;

    void Awake()
    {
        icon = GetComponentsInChildren<Image>()[1];
        icon.sprite = data.itemIcon;
    
        // .. GetComponents의 순서는 계층구조의 순서 -> Hirarchy 뷰 상의 순서
        Text[] texts = GetComponentsInChildren<Text>();
        textLevel = texts[0];
        textName = texts[1];
        textDesc = texts[2];
        textName.text = data.itemName;
    }

    void OnEnable()
    {
        textLevel.text = "Lv." + (level + 1);

        switch(data.itemType)
        {
            case ItemData.ItemType.Melee:
            case ItemData.ItemType.Range:
                textDesc.text = string.Format(data.itemDesc, data.damages[level] * 100, data.counts[level]);
                break;
            case ItemData.ItemType.Glove:
            case ItemData.ItemType.Shoe:
                textDesc.text = string.Format(data.itemDesc, data.damages[level] * 100);
                break;
            default:
                textDesc.text = string.Format(data.itemDesc);
                break;

        }
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

- text가 3개이므로 각각을 초기화합니다. 저번에 배열형태로 초기화 했던 이유가 이것입니다. 배열의 index는 유니티에서의 계층순서입니다.
- 아이템 이름 text는 변경되거나 하지 않으므로, 초기화와 동시에 지정해줍니다.
- LateUpdate() 함수에서 작성했던 레벨은 이제 랜덤하게 등장하는 능력치업그레이드에 따라 나올것이므로 OnEnable()함수에서 작성합니다.

![image](/images/2024/2024-01-31/capture_7.png)

