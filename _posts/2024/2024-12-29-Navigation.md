---
layer: single
title: "유니티 Navigation - NavMeshAgent"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

이번 포스팅에서는 유니티에서 최단 거리를 탐색하고 타깃을 추적할 수 있는 컴포넌트인 NavMeshAgent에 대해
정리해보았다.



## 개념

유니티의 Navigation 시스템을 사용하여 씬 내에서 지능적으로 탐색 및 이동할 수 있는 캐릭터를 만들수있다.

**NavMesh**는 자동으로 씬에 생성되며, 게임월드에서 "걸을 수 있는 표면"을 뜻한다.

이를 이용해 목적지까지 경로를 찾고 이동하는 작업을 수행하는 컴포넌트가 NavMeshAgent 이다.


## 내부 동작

Navigation을 사용하여 캐릭터를 움직이려면 크게 두가지 기능으로 나뉜다.

![image](/images/2024/2024-12-29/capture_1.PNG) 

1. 맵을 탐색하여 목적지까지 경로를 추론 - Global Navigation
2. 해당 목적지까지 이동 - Local Navigation

![image](/images/2024/2024-12-29/capture_2.PNG) 


게임 씬에서 두 지점 사이의 경로를 찾기 위해서는 시작위치와 목적지위치를 가장 가까운 폴리곤에 매핑한다.

다음, 시작 위치에서 탐색을 시작하여 모든 이웃 폴리곤을 거쳐 목적지 폴리곤에 도달한다.

이때 Navigation은 경로를 찾는 알고리즘인 **"A*"**를 사용한다.

![image](/images/2024/2024-12-29/capture_3.PNG) 


시작 폴리곤에서 목적지 폴리곤까지 경로를 정의하는 폴리곤의 시퀀스를 Corridor 라고 한다.

Agent는 Corridor를 따라 보이는 가장 가까운 코너를 향해 움직이는 방법으로 목적지에 도달하게된다.


## 사용방법

Navigation 시스템은 에디터 상단의 Window - AI - Navigation을 누르게되면 Navigation 창이 생긴다.

맵으로 사용할 오브젝트를 선택한 뒤, Navigation Static을 체크해준다.

![image](/images/2024/2024-12-29/capture_4.PNG) 


캐릭터가 이동할 수 있는 오브젝트는 Navigation Area를 Walkable로 선택하고,

이동이 불가능하게 할 오브젝트는 Not Walkable로 바꿔준다.

![image](/images/2024/2024-12-29/capture_5.PNG) 

Bake 탭에서 Bake 버튼을 눌러주면 씬에서 하늘색의 NavMesh가 자동으로 생성된것을 확인할 수 있다.

이제 이동시킬 캐릭터에 NavMeshAgent 컴포넌트를 추가하고, Test 스크립트를 추가한다.

```c#
using UnityEngine;
using UnityEngine.AI;

public class Test : MonoBehaviour
{
    private NavMeshAgent agent;
    public transform target;

    private void Start()
    {
        nav = GetComponent<NavMeshAgent>();
    }

    private void Update()
    {
        if(Input.GetMouseButtonDown(0))
            nav.SetDestination(target.position);
    }
}
```

- NavMeshAgent를 사용하기위해서는 UnityEngine.AI 네임 스페이스를 추가해주어야한다.
- SetDestination 메서드를 사용하여 목적지로 이동하도록하였다.


## NavMeshAgent 컴포넌트

NavMeshAgent 컴포넌트의 속성값들을 조정하여 캐릭터의 세부적인 움직임을 설정해줄수있다.

![image](/images/2024/2024-12-29/capture_6.PNG) 

* **Agent Type** : 타입에 따라 갈 수 있는 경로를 다르게 하기위한 설정
* **Base Offset** : NavMeshAgent의 충돌 실린더와 해당 컴포넌트가 부착된 오브젝트의 중심점을 맞추기위한 속성
* **Speed** : 최대 이동 속도
* **Angular Speed** : 최대 회전 속도
* **Acceleration** : 최대 가속
* **Stopping distance** : 목적지 도착전 어느정도의 거리에서 멈출지 설정값
* **Auto Breaking** : 활성화시 에이전트는 목적지에 다다를 때 속도를 줄여서 목적지에 정확하게 도착
* **Radius** : Agent의 장애물 회피 반경 반지름
    - 다른 Agent들과의 충돌 반경을 나타낸다.
* **Height** : 장애물을 통과하기위한 Agent의 높이
* **Quality** : 장애물의 회피 품질로, 높을수록 경로가 더 세밀해진다.
    - Quality가 높을수록 CPU의 사용량이 많아진다.
* **Priority** : 다른 Agent가 경로에 있다면 우선순위가 낮은 Agent는 회피대상에서 제외된다.
    - Priority가 낮을 수록 우선순위가 높다.
* **Auto Traverse Off Link** : OffMeshLink를 자동으로할지 여부
    - 애니메이션을 사용한다면 비활성화해야한다.
* **Auto Repath** : 기존의 경로가 잘못된 경우 Agent가 새로운 경로찾기를 시도할지에 대한 여부
* **Area Mask** : Agent가 지나다닐 수 있는 Area를 따로 설정할 수 있다.


### NavMeshAgent 함수

![image](/images/2024/2024-12-29/capture_7.PNG) 


## NavMeshObstacle 컴포넌트

동적인 장애물을 만들때에 사용하는 컴포넌트이다.

NavMeshObstacle 컴포넌트가 부착된 장애물을 생성하여, Agent의 이동경로에 놓게되면 장애물에 가로막혀 움직이지
못하는 상황을 볼 수 있다.

![image](/images/2024/2024-12-29/capture_8.PNG) 


이때 Carve 속성을 체크하면, NavMesh가 동적으로 수정되면서 Agent가 다른 경로를 찾아갈 수 있다.

- Move Threshold : 값만큼 장애물이 움직인다면 다시 Carve를 진행한다.
- Time To Stationary : 장애물이 멈췄을 때, 어느정도 시간뒤에 Carve를 다시 진행할 지 설정한다.
- Carve Only Stationary : 장애물이 멈추지 않아도 Carve가 진행되도록 할것인지 여부
    - Carve 작업이 자주 실행되면 성능에 좋지않은 영향을 끼칠 수 있다.


출처 - https://sikpang.tistory.com/7