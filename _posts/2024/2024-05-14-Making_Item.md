---
layer: single
title: "아이템만들기 - Light & Particle System"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 목표

- 플레이어가 사용할 무기, 코인, 총알 등 아이템 만들기

![image](/images/2024/2024-05-14/capture_1.gif)

- 아이템에 Light, Particle를 추가해서 좀 더 멋진 아이템을 만들어보자.


## 준비

- 준비된 리소스는 에셋스토어에 골드메탈님이 무료로 올려주신것을 가져왔습니다.

![image](/images/2024/2024-05-14/capture_2.PNG)


- 먼저, 씬에 무기아이템 Hammer를 올려놓습니다.

![image](/images/2024/2024-05-14/capture_3.PNG)

- Hammer 오브젝트에 Create Empty를 만들어 Light와 Particle 이름으로 추가해줍니다.

![image](/images/2024/2024-05-14/capture_4.PNG)

### Light 추가

- Light를 선택하고, Add Component에서 Light를 추가합니다.

![image](/images/2024/2024-05-14/capture_5.PNG)

- Type은 Point, Color는 스포이드모양을 클릭에서 Hammer의 색을 뽑아줍니다.
- Range는 범위 Intensity는 강도로, 원하는 정도로 적당히 조절하면 됩니다.
- Shadow로 그림자를 추가할 수 있지만 여기에서는 사용하지 않습니다.

![image](/images/2024/2024-05-14/capture_6.PNG)

- 뭔가 조금 영롱해진 느낌입니다.


### Particle 추가

- 파티클은 작고 단순한 이미지 또는 메시로, 연기같은 연출을 도와줍니다.
- Particle 오브젝트를 선택하고, Add Component에서 Particle System을 추가합니다.

![image](/images/2024/2024-05-14/capture_7.PNG)

![image](/images/2024/2024-05-14/capture_8.gif)

- Particle System에 여러가지 효과를 줄 수 있는 옵션들이 있는데, 6가지 정도만 사용해봅니다.

1. 가장먼저 입체의 모양을 정해줍니다. Renderer탭에 Material부분에서 기본적으로 제공하는 Defalut line을 사용합니다.

2. 파티클이 나오는 양을 조절해줍니다. Emission 탭의 Rate over Time를 조절하여 원하는 정도의 양을 정해줍니다.

3. 파티클이 퍼져나오는 모양을 정해줍니다. Shape탭에서 모양과 각도, 위치까지 정할 수 있습니다.

4. 파티클이 퍼져나가는 속도를 제어해줍니다. Limit Velocity over Lifetime 탭에서 속도를 제어할 수 있습니다.

5. 파티클 색깔을 정해줍니다. Color over Lifetime탭에서 색깔 및 알파값도 제어할 수 있습니다.

6. 파티클 사이즈를 정해줍니다. Size over Lifetime탭에서 파티클 크기가 어떤식으로 변해갈 지 정할 수 있습니다.


![image](/images/2024/2024-05-14/capture_9.gif)

- 입맛에 맞게 파티클 시스템을 잘 만진다면 멋진 파티클 이펙트를 만들 수 있을것같습니다.


### 회전효과

- 아이템이 가만히 있으니 허전한것 같아 추가해봤습니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Item : MonoBehaviour
{
    public enum itemType { Ammo, Coin, Grenade, Heart, Weapon};
    public itemType type;
    public int value;

    void Update()
    {
        transform.Rotate(Vector3.up * 20 * Time.deltaTime);
    }
}

```

- Hammer의 Freeze Rotation X, Z 옵션을 활성화 시켜준 뒤 스크립트를 붙여 실행해봅니다.

![image](/images/2024/2024-05-14/capture_10.gif)
