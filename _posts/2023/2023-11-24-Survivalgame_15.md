---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[14]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 아이템 데이터 만들기
- 게임에서 사용될 아이템 그리고 무기 데이터를 관리할 수 있도록 스크립트를 제작해보겠습니다.

## ItemData.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// ... 커스텀 메뉴를 생성하는 속성
[CreateAssetMenu(fileName ="Item",menuName ="Scriptable Object/ItemData")]
// ... 아이템(무기) 관련 스크립트
public class ItemData : ScriptableObject
{
    // .. 근접 원거리 글러브 신발 힐
    public enum ItemType { Melee, Range, Glove, Shoe, Heal}

    [Header("# Main Info")]
    public ItemType itemType;
    public int itemId;
    public string itemName;
    public string itemDesc;
    public Sprite itemIcon;

    [Header("# Level Data")]
    public float baseDamage;
    public int baseCount;
    public float[] damages;
    public int[] counts;

    [Header("# Weapon")]
    public GameObject projectile;

}
```

- 우선 스크립트의 타입을 Monobehavior가 아닌, ScriptableObject로 변경합니다. 이 클래스는 데이터를 저장하는 데 사용할 수 있는 데이터 컨테이너입니다.
- 그리고 속성으로 CreateAssetMenu를 주어 메뉴를 생성할 수 있도록 했습니다.
- 아래는 게임에서 사용될 데이터들을 정리해놓은것입니다.


## 데이터 관리 애셋
- 프로젝트 폴더 아래에 Data 라는 새로운 폴더를 만들어줍니다.
- Data-Create에서 맨위를 보면 Scriptable이라는 새로운 목록이 있습니다. 선택합니다.
- 아래와 같이 아이템별 정보를 설정할 수 있습니다.

![image](/images/2023-11-24/capture_1.png)

- 이런 방식으로 Scriptable 오브젝트를 아이템마다 생성합니다.

![image](/images/2023-11-24/capture_2.png)


# 레벨업 버튼 UI 만들기

- 무기강화를 위한 레벨업 버튼 UI입니다.
- Canvas 아래에 Create Empty를 생성하여 LevelUp으로 이름을 바꿔줬습니다.
- LevelUp 아래에 Text UI와 Image UI를 붙여 아래처럼 설정했습니다.


![image](/images/2023-11-24/capture_3.png)


- 만든 UI를 복사하여 Item 0부터 Item 4까지 각각 설정해줍니다.

![image](/images/2023-11-24/capture_4.png)

- LevelUp 오브젝트에 AddComponent - Vertical Layout Group를 붙여주게되면 화면상에 세로로 차곡차곡 나타나는것을 볼 수 있습니다.

![image](/images/2023-11-24/capture_5.png)


## Item.cs

- Item UI를 관리하고, 이벤트를 처리하는 스크립트입니다.

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

- Image와 Text는 Item 0 ~ Item 4 오브젝트 아래의 컴포넌트들 이므로 GetComponentsInChildren로 초기화합니다.
- Text UI는 나중에 더 추가될 수 있으므로 배열로 선언해줍니다.
- UI 버튼들을 클릭했을 때 이벤트를 처리하는 함수로 OnClick 함수를 만들었습니다. 버튼을 누르면 레벨을 한단계 올리고, 최대레벨에 도달했을 경우에 버튼을 비활성화 시켰습니다.
- 저장한 뒤 Item 0 ~ Item 4 별로 Item 스크립트를 붙이고 Onclick 함수가 동작하도록 설정해줍니다.

![image](/images/2023-11-24/capture_6.png)


![image](/images/2023-11-24/capture_7.gif)
