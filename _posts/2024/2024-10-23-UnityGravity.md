---
layer: single
title: "유니티 중력 시스템[Rigidbody]"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


## Intro
&nbsp;
이번 포스팅에서는 유니티엔진을 사용할 때 필수적으로 알아야할 컴포넌트인 Rigidbody에 대해서
찾아보고, 정리하였습니다.  
&nbsp;
게임에서 물리적인 상호작용은 게이머가 게임에 몰입하는데 아주 중요한 역할을합니다.
물체가 여기저기 부딪히고, 떨어지는모습이나 캐릭터를 조작할 때 중력의힘에 영향을 받는 등 플레이어가 좀 더
실감나는 게임을 플레이할 수 있도록 해줍니다. 
&nbsp;

유니티에서는 이런 물리적 효과를 구현할 수 있도록 도와주는 물리엔진을 제공합니다. 

### Rigidbody 컴포넌트

&nbsp;
**Rigidbody**는 게임 오브젝트에 물리법칙을 적용하는 핵심 컴포넌트중 하나입니다. Rigidbody 컴포넌트가
붙은 오브젝트는 중력의 영향을 받고, 충돌이나 힘의 작용을받아 마치 현실에 있는것과같은 움직임을 보이게됩니다.

&nbsp;
Rigidbody는 에디터의 인스펙터창에서 추가할 수 있습니다.  
큐브 오브젝트를 하나 생성한 뒤, Rigidbody 컴포넌트를 붙여보았습니다.  
&nbsp;
![image](/images/2024/2024-10-23/capture_1.PNG) 
&nbsp; 
Rigidbody 컴포넌트에서 제어할 수 있는 옵션들을 정리해봤습니다.  

- **Mass(질량)** : 객체의 <u>질량</u>입니다. Default값은 1로, 질량은 중력의 영향을 받는 속도에는 영향을        주지는 않지만 힘과 충돌 시 반응에 영향을 미칩니다.
- **Drag(항력)** : 객체가 <u>이동할 때 받는 공기 저항</u>입니다. 0은 저항이 없음을 나타내고, 높은값일수록 큰 저항을 받게됩니다.
- **Angular Drag(각 항력)** : 객체가 <u>회전할 때 받는 공기 저항</u>입니다. 높은값일수록 더 빨리 멈춥니다. 
- **Use Gravity(중력 사용여부)** : 활성화되어있으면 객체가 중력의 영향을 받습니다.
- **Is Kinematic(Kinematic 여부)** : 활성화되어있으면 객체는 물리 엔진에의해 구동되지 않고 Transform으로만 조작할 수 있습니다. 
- **Interpolate(보간)** : 객체의 움직임이 끊기는것을 방지하기위한 보간옵션입니다.
- **Collision Detection(충돌 감지)** : 빠르게 움직이는 객체가 다른 객체를 통과하지 않도록 충돌감지를 설정합니다.
- **Constraints(제약)** 
    - **Freeze Position** : 체크되어있는 축에서의 객체의 이동을 막습니다.
    - **Freeze Rotation** : 체크되어있는 축에서의 객체의 회전을 막습니다. 


&nbsp; 
&nbsp; 

### 전역 중력
&nbsp; 
Project Settings에서 중력을 조정할 수 있습니다.  
&nbsp; 
![image](/images/2024/2024-10-23/capture_2.PNG) 
&nbsp; 

- 3D/2D 프로젝트에서 기본적으로 Y축에 -9.81로 중력값이 설정되어있습니다. 
&nbsp; 


#### 2D 프로젝트와 3D 프로젝트 중력 설정 차이

- **3D 프로젝트** : 중력이 X, Y, Z축 모두에 적용되며, 기본적으로 중력이 아래쪽으로 작용하도록 되어있습니다.  
- **2D 프로젝트** : 중력이 XY 평면에서만 작용하며 기본적으로 Y축에만 적용되도록 설정되어있습니다. 따라서 2D 캐릭터는 위아래로만 중력의 영향을 받게됩니다.  

&nbsp; 
&nbsp; 

전역 중력값을 다르게 설정하면 다양한 환경을 시뮬레이션할 수 있습니다. 
&nbsp; 

- 지구 : -9.81(Default)
- 달 : 약 -1.62
- 화성 : 약 -3.71
- 목성 : 약 -24.79

&nbsp; 

### 개별 오브젝트의 중력 조절

유니티에서 개별 오브젝트의 중력을 조절하는 방법은 두가지입니다. 
&nbsp;

첫째로 Rigidbody의 Use Gravity 속성을 활용하는 방법입니다.  
&nbsp;
Use Gravity 속성을 통해 중력의 영향을 받을지의 여부를 설정할 수 있습니다. 
인스펙터창에서 체크를 통해 설정할 수 있으며, 스크립트에서는 useGravity 메서드를 사용하여 컨트롤합니다.  

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GravityTest : MonoBehaviour
{
    private Rigidbody rigid;

    private void Awake()
    {
        rigid = GetComponent<Rigidbody>();    
    }

    private void Update()
    {
        // 스페이스바 입력을 통해 중력사용 토글
        if(Input.GetKeyDown(KeyCode.Space))
        {
            rigid.useGravity = !rigid.useGravity;
        }
    }
}
```

&nbsp;

특정 방향으로 중력을 적용하거나 오브젝트에 따라 중력값을 추가할 수 있습니다. 

```c#
private void Update()
{
        Vector3 dirGravity = new Vector3(-1f, 0, 0);        // 왼쪽으로 중력 적용
        rigid.AddForce(dirGravity * 9.81f);
}
```

### 다양한 Rigidbody 옵션

Rigidbody 컴포넌트를 통해 중력뿐만아니라 다양한 옵션들을 스크립트를 통해 설정할 수 있습니다.  


![image](/images/2024/2024-10-23/capture_3.PNG) 


&nbsp;

이런 옵션들을 적절히 사용한다면 물체의 여러 재미있는 움직임들을 표현할 수 있고, 플레이어로 하여금 캐릭터를 컨트롤하는 재미를 느낄수 있도록 할 수 있습니다.